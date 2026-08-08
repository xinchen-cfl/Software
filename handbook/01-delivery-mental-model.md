# 从本地代码到生产：交付心智模型

## 1. 一条发布链究竟改变了什么

软件交付不是一个模糊的“上线”按钮，而是一组可以观察的状态变化：

```text
本地工作区产生改动
→ Git commit 保存代码快照
→ PR 把补丁带到共享分支
→ CI 在干净环境复现检查
→ 构建系统生成不可变运行产物
→ staging 运行并验证该产物
→ production 运行同一个产物
→ 必要时重新选择旧产物回滚
```

每一步都应该回答三个问题：

1. 输入状态是什么？
2. 这一步改变了什么？
3. 什么证据证明结果符合预期？

## 2. Git 中容易混淆的对象

### 工作区、worktree、branch 和 commit

- 工作区是你看到和编辑的文件。
- worktree 是一个受 Git 管理的独立检出目录。
- branch 是指向某个 commit 的可移动指针。
- commit 是不可变的代码快照，并通过 parent 形成历史链。

创建 branch 不会复制代码；创建 worktree 才会得到另一个目录。同一 branch 通常只在一个 worktree 中检出。

真正危险的不是“旧 branch”，而是：

```text
未提交 + 未推送 + 没人知道属于哪个状态的改动
```

只要工作已经 commit，它通常可以 rebase、cherry-pick 或恢复。

### fetch 和 pull

`git fetch` 更新本地对远端状态的认识，不修改当前检出的文件。`git pull` 在 fetch 后继续把远端历史整合进当前 branch。

因此，没有 pull 并不代表 Git 不知道远端发生了变化。

### merge、squash 和 rebase

- merge 保留分支历史并增加 merge commit。
- squash 把整个 PR 的补丁压成目标分支上的一个新 commit。
- rebase 把提交重新建立在新 base 上，因此 commit SHA 会改变。

SHA 不同不一定代表代码不同。Squash 或 rebase 后，可用 patch 比较或 `git cherry` 判断等价改动。

## 3. 测试不是一个统一动作

测试工具可以通用，测试内容必须针对项目编写。Python 项目可能使用 pytest，但 pytest 不会自动知道业务正确答案。

不同检查提供不同证据：

```text
focused test  检查当前行为
status/diff   检查准备提交的实际范围
local fence   检查本机最终补丁
PR CI         在干净统一环境复现
image build   检查能否形成运行产物
staging       检查产物在生产式环境的集成结果
production    检查真实环境是否运行目标产物
```

如果代码、依赖、配置、生成文件和环境假设都没变化，立即重复同一个本地测试通常没有新增价值。跨越独立环境或状态边界时，即使执行相同测试，也可能产生新的证据。

## 4. CI 与 CD

CI 负责把检查自动化并留下共享证据，例如：

- 安装声明的依赖；
- 运行测试、lint、类型检查；
- 从合并后的 commit 构建镜像；
- 把产物保存到 registry。

CD 负责改变运行环境，例如：

- 拉取指定镜像；
- 替换 staging 或 production 容器；
- 执行迁移和健康检查；
- 失败时回滚。

CI 绿色不等于 production 已更新。它只证明检查或构建阶段成功。

### 为什么本地测试后还运行 CI

本地测试回答“当前电脑和工作区能否通过”；CI 回答“GitHub 上真正提交的代码能否在全新标准环境中通过”。CI 可以发现：

- 本地存在但未提交的文件；
- 虚拟环境中未声明的依赖；
- macOS 与 Linux 的文件名或平台差异；
- 本地缓存或环境变量造成的假通过；
- 与最新 base 合并后的问题。

如果 PR CI 可以被管理员绕过，那么合并后的构建流程保留一次最小测试很有价值：坏代码可以进入 main，但不应继续生成发布镜像。

## 5. 镜像、容器、tag 与 digest

- 镜像是可重复启动的运行模板。
- 容器是镜像的一次运行实例。
- tag 是方便人识别的可移动标签。
- digest 是镜像内容的不可变指纹。

生产发布应该记录 digest：

```text
source commit -> image digest -> running container
```

staging 和 production 不应分别构建镜像。正确方式是构建一次，然后把 staging 验证过的同一个 digest 晋级到 production。

GitHub Actions 中 `.dockerbuild` 构建记录也可能有自己的 digest；那是记录文件的指纹，不是容器镜像 digest。部署时应使用 registry 中的镜像 digest。

## 6. staging 与 production

staging 是真实部署，但不承接真实用户流量。它应尽可能复现 production 的运行方式，用来发现镜像启动、环境变量、依赖、网络、代理和架构问题。

production 是承接真实流量和数据的运行环境。

仅创建两个目录并不能形成隔离。隔离需要结合：

- 独立进程或容器；
- Compose project、网络和端口；
- 配置、Secret 和权限；
- 数据库和存储；
- 主机、集群或云账号；
- 入口与流量控制。

训练项目可以把 staging 与 production 放在同一服务器，但高风险业务通常需要更强隔离。

## 7. 回滚

应用回滚不是撤销 Git，也不是重新构建旧代码。它是让 production 重新运行一个已经存在、验证过的旧 artifact。

最小状态记录：

```text
current artifact
actual running artifact ID
previous artifact
artifact withdrawn by rollback
```

单级 `previous-image` 足够覆盖最基本回滚。需要回到更早版本时，再引入发布历史或 release ledger。

应用镜像回滚不自动等于数据库回滚。涉及 schema 和数据变化时，需要单独设计向前兼容迁移、备份和恢复策略。

## 8. GitHub Releases 不是发布列车

GitHub Releases 是可选的版本公告与下载页面，通常与 `v1.0.0` tag、发行说明和安装包关联。

一个项目即使没有 GitHub Release，也可以通过 Actions、container registry 和部署系统完成真实生产发布。不要把以下概念混为一谈：

- 业务中的 `/release` 接口；
- 软件 release process；
- GitHub Releases 页面；
- GHCR Packages 中的运行镜像。
