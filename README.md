# 软件工程交付

这个仓库独立保存从本地开发到生产发布的软件工程学习与复习资料：

```text
handbook/   给人阅读、复习和继续补充
```

本仓库不参与 Harness 安装、运行或规则同步，也不再提供可执行 AI Skill。实战研发能力统一在私有 Harness 仓库维护；这里的内容仅供人学习，不构成 Agent 当前行为规则。

## 复习路线

1. [`01-delivery-mental-model.md`](handbook/01-delivery-mental-model.md)：先理解所有对象与边界。
2. [`02-tradeploy-case-study.md`](handbook/02-tradeploy-case-study.md)：复盘一次真实的本地、GitHub、Linux 服务器实践。
3. [`03-release-checklists.md`](handbook/03-release-checklists.md)：实际工作时快速检查。
4. [`04-zero-project-foundation.md`](handbook/04-zero-project-foundation.md)：空项目应该建立哪些最小工程基础。

## 核心原则

```text
合并后的提交
→ 持续集成只构建一次
→ 不可变产物摘要
→ 预发布验证
→ 同一个摘要进入生产
→ 已知且可用的回滚目标
```

仓库只记录可公开复用的工程知识，不保存服务器密码、Token、私钥或生产密钥。
