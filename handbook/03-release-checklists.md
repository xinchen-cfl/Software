# 软件交付检查表

## 开发前

- [ ] 阅读仓库指令和现有流程
- [ ] 确认 base branch 与 commit
- [ ] 检查 status、branch 和 worktree
- [ ] 确认没有覆盖未知改动

## 提交前

- [ ] 相关测试针对正确原因先失败、后通过
- [ ] status 只包含预期文件
- [ ] diff 没有秘密、调试代码、误删或无关改动
- [ ] 项目要求的 lint/type/build/test 已通过

## 拉取请求与持续集成

- [ ] commit 和 push 成功
- [ ] PR base/head 正确
- [ ] CI 在干净环境通过
- [ ] rebase/冲突处理后重新验证受影响内容
- [ ] 识别合并后真正进入目标分支的 commit

## 运行产物

- [ ] artifact 来自合并后的目标分支 commit
- [ ] 平台与运行环境兼容
- [ ] 记录完整 digest，不使用 `latest`
- [ ] registry 中可以拉取该 digest

## 预发布

- [ ] 记录部署前 artifact
- [ ] health/readiness 通过
- [ ] version 对应 source commit
- [ ] 配置 artifact 与实际 artifact ID 一致
- [ ] 本次关键功能 smoke test 通过
- [ ] production 未发生变化

## 生产

- [ ] 明确 production 授权和影响范围
- [ ] 已记录 rollback target
- [ ] 使用 staging 验证过的同一个 digest
- [ ] health/readiness/version 通过
- [ ] 关键业务行为通过
- [ ] 共享主机上的相关服务未受影响

## 回滚

- [ ] previous artifact 存在且可获取
- [ ] 记录被撤回 artifact
- [ ] 旧 artifact 成功启动
- [ ] 健康、版本、身份和关键行为重新验证
- [ ] 数据库变化单独评估

## 清理

- [ ] fetch/prune 最新远端状态
- [ ] 确认补丁已经进入目标分支
- [ ] 所有 worktree 无未知 dirty/untracked 文件
- [ ] 删除不再使用的 worktree
- [ ] 删除不再使用的本地 branch
- [ ] 获得授权后再删除远端 branch
