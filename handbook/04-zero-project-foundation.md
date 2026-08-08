# 空项目的软件工程最小基础

## 1. 目标

空项目不需要一次性复制大型公司的全部工程体系。第一目标是建立最小闭环：

```text
代码可以复现运行
→ 改动可以检查
→ 提交可以追踪
→ 共享环境可以复现
→ 运行产物可以识别
→ 失败可以恢复
```

## 2. 第一批必须建立的内容

### 版本控制

- 初始化 Git 和默认分支；
- 忽略秘密、缓存、构建产物和本地环境；
- 新任务使用专属功能分支；
- 并发或目录隔离确有需要时才使用工作树。

### 可复现入口

- 明确依赖声明和安装命令；
- 明确本地启动命令；
- 需要构建时明确构建命令；
- 不依赖某个人机器中未声明的包、缓存或环境变量。

### 最小检查

- 给核心路径留下一个能失败的检查；
- 测试层级匹配风险，能在低层证明就不升级到端到端；
- 状态与差异检查补足测试看不到的修改范围；
- 持续集成在干净环境复现必要检查。

### 可识别产物

需要部署时，从合并后的源代码构建一次，并记录不可变身份。预发布和生产使用同一产物，不分别重建。

## 3. 按真实风险增加能力

### 出现秘密或外部凭据

- 禁止进入代码、日志、客户端产物和镜像层；
- 使用最小权限和短生命周期；
- 明确负责人、轮换和撤销路径；
- 泄漏后在提供方撤销或轮换，不能只删除代码。

### 出现持久数据

- 建立备份和真实恢复验证；
- 明确可接受数据损失和恢复时间；
- 迁移检查锁表、前后兼容和数据恢复；
- 区分应用回滚与数据恢复。

### 出现关键用户路径

- 用少量端到端测试覆盖集成风险；
- 测试数据相互隔离；
- 等待可观察条件而不是固定睡眠；
- 波动测试定位根因，不靠重试刷绿。

### 出现真实生产流量

- 建立健康、错误、延迟和关键业务信号；
- 明确生产验证和回滚目标；
- 风险需要时再使用渐进式发布、服务等级目标和容量规划；
- 事故后记录影响、时间线、止损、恢复和系统性改进。

## 4. 不应提前建立的内容

```text
没有数据库       → 不建立迁移和备份系统
没有并发开发     → 不强制 worktree
没有容器需求     → 不强制 Docker
没有生产部署     → 不虚构 staging/production
没有真实可靠性目标 → 不建立复杂 SLO 和告警体系
没有规模证据     → 不引入微服务和复杂编排
```

## 5. 参考 Agent

本章提炼自 Agency Agents 中与软件工程最相关的专项角色：

- [Git Workflow Master](https://github.com/msitarzewski/agency-agents/blob/main/engineering/engineering-git-workflow-master.md)
- [Minimal Change Engineer](https://github.com/msitarzewski/agency-agents/blob/main/engineering/engineering-minimal-change-engineer.md)
- [Test Automation Engineer](https://github.com/msitarzewski/agency-agents/blob/main/testing/testing-test-automation-engineer.md)
- [Database Reliability Engineer](https://github.com/msitarzewski/agency-agents/blob/main/engineering/engineering-database-reliability-engineer.md)
- [Secrets & Credential Hygiene Engineer](https://github.com/msitarzewski/agency-agents/blob/main/security/security-secrets-credential-engineer.md)
- [SRE](https://github.com/msitarzewski/agency-agents/blob/main/engineering/engineering-sre.md)

只吸收可迁移原则，不复制角色人格、固定覆盖率、固定流量倍数或默认重型流程。
