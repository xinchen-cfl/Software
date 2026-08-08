# Software Engineering Delivery

这个仓库沉淀从本地开发到生产发布的软件工程经验，分为两个入口：

```text
skill/      给 Codex 直接安装和执行
handbook/   给人阅读、复习和继续补充
```

## 直接使用 Skill

技能位于 [`skill/ship-software-safely`](skill/ship-software-safely)。它用于让 Codex 在真实项目中执行一条可追踪的软件交付链：

```text
开发 → Git/PR → CI → 不可变产物 → staging → production → rollback → cleanup
```

## 复习路线

1. [`01-delivery-mental-model.md`](handbook/01-delivery-mental-model.md)：先理解所有对象与边界。
2. [`02-tradeploy-case-study.md`](handbook/02-tradeploy-case-study.md)：复盘一次真实的本地、GitHub、Linux 服务器实践。
3. [`03-release-checklists.md`](handbook/03-release-checklists.md)：实际工作时快速检查。

## 核心原则

```text
merged commit
→ CI build once
→ immutable artifact digest
→ staging verification
→ same digest in production
→ known rollback target
```

仓库只记录可公开复用的工程知识，不保存服务器密码、Token、私钥或生产密钥。

