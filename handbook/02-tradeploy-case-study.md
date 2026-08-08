# Tradeploy 实战复盘

## 1. 实战目标

使用一个小型 FastAPI 服务，把看似黑盒的开发和上线过程变成可以解释、验证和恢复的链路。训练重点不是业务功能，而是交付边界。

## 2. 最终系统

```text
macOS 本地仓库
├── feature branch / worktree
├── shared Python venv
└── pytest

GitHub
├── PR CI
├── main image workflow
└── GHCR immutable image

Linux server
├── tradeploy-staging     127.0.0.1:18080
├── tradeploy-production  127.0.0.1:28080
└── deploy / inspect / rollback scripts
```

staging 和 production 使用独立 Compose project、容器、网络、端口和状态目录。它们运行同一个镜像 digest，但不是同一组容器。

## 3. 从空仓库到可运行服务

第一阶段建立了：

- FastAPI 应用；
- `/health`、`/ready`、`/version` 等可观测接口；
- pytest 测试；
- requirements；
- `.gitignore`；
- Dockerfile 和 `.dockerignore`。

曾经出现 `ModuleNotFoundError: app`。根因不是测试断言，而是 Python 的统一入口与模块搜索路径。使用虚拟环境中的 Python 运行 `python -m pytest`，可以明确解释器和模块入口。

共享 venv 可以服务多个 worktree，但依赖必须兼容。同一项目的 worktree 通常共享依赖更快；需要测试不同依赖组合时才拆分环境。worktree 隔离代码目录，不等于虚拟环境或容器隔离。

## 4. Git 和 worktree 实践

每个并发功能可以拥有自己的 branch 和 worktree。功能完成后：

```text
测试 → diff 围栏 → commit → push → PR
→ 必要时 rebase → merge → fetch/pull main
→ 检查 dirty files → remove worktree → delete branch
```

经验：

- `git status` 发现修改范围；
- `git diff --check` 发现尾随空格和换行问题；
- `git diff` 检查实际准备提交的内容；
- 测试通过不能替代 diff；
- Squash merge 后使用 patch 等价性判断，不应只比较 commit SHA；
- rebase 后使用 `--force-with-lease` 更新远端分支。

## 5. 镜像实践

最初手动经历了完整底层流程：

```text
docker build
→ 本地运行容器
→ 验证 health/version
→ 针对 linux/amd64 构建
→ docker save
→ checksum
→ scp
→ server docker load
```

这一步的价值是理解自动化背后的真实动作。日常流程随后收敛为：

```text
merge main
→ GitHub Actions test
→ build linux/amd64
→ push GHCR
→ output digest
→ server pull digest
```

因此自动化不是魔法，只是把已经理解并验证过的手动步骤固定下来。

## 6. 服务器实践

服务器保留原有服务，并新增 Docker 与独立目录。应用入口仅绑定回环地址，避免未经授权暴露公网。

部署配置包含：

- app 容器；
- Nginx 反向代理；
- healthcheck；
- staging 与 production 的独立 project name 和 host port；
- `pull_policy: never`，由部署脚本显式 pull 指定 artifact。

服务器侧不进行业务开发。它只拉取并运行 GitHub registry 中已经构建的镜像。

## 7. CI 与部署脚本

PR workflow 运行快速测试，main workflow 在测试通过后构建并推送镜像。PR 检查是否阻止合并，由 GitHub ruleset 或 branch protection 决定；仅存在 workflow 不代表 merge 被强制阻塞。

部署脚本负责：

1. 校验 staging/production 参数；
2. 要求完整 sha256 digest；
3. 拉取目标镜像；
4. 记录旧 artifact；
5. Compose 更新并等待健康；
6. 检查 health、ready、version；
7. 核对配置镜像与实际运行镜像；
8. 更新当前状态文件。

回滚脚本读取 production 的 previous artifact，重新部署并重复验证。检查脚本同时报告两个环境，即使其中一个失败也尽量继续收集证据，最终返回非零状态。

## 8. 完整毕业演练

最终独立变更新增了 `/release` 接口：

```text
创建 feature branch
→ 先写失败测试并观察 404
→ 实现最小接口并得到 5 passed
→ status/diff 围栏
→ commit/push/PR
→ PR CI
→ squash merge
→ main workflow 构建新 digest
→ staging 部署和功能 smoke test
→ production 部署同一 digest
→ production 回滚到 previous digest
→ staging 保持新版本
→ production roll forward 到新 digest
→ 清理 branch/worktree
```

最终能明确回答：

- 当前 source commit 是什么；
- registry 中部署 artifact 的 digest 是什么；
- staging 和 production 分别运行什么；
- 健康和关键业务行为是否通过；
- production 的 rollback target 是什么；
- 本地是否还存在未知工作。

## 9. 尚未覆盖的生产能力

当前实践建立的是单服务交付基础，尚未覆盖：

- 域名、TLS 和真实公网流量；
- 数据库迁移、备份与数据恢复；
- Secret 管理；
- 日志、指标、告警与 SLO；
- 零停机、canary、蓝绿或滚动发布；
- 自动回滚条件；
- 镜像签名、SBOM 与供应链安全；
- 多服务依赖和发布顺序；
- 灾难恢复与服务器替换。

这些能力应随真实风险增加，而不是为了形式一次性全部引入。
