
# 一、初始化项目设置

## 1. 保护 main 分支
```bash
# 在 GitHub/GitLab 中设置分支保护规则
- 禁止直接 push 到 main
- 要求 PR 至少 1 人审批
- 合并前必须通过 CI
```

## 2. 本地配置

```bash
# 设置默认分支名
git config --global init.defaultBranch main

# （可选）安装 commitizen 工具辅助规范提交
npm install -g commitizen
```

# 二、日常工作流程

## 场景 1：开始一个新功能

```bash
# 1. 同步最新的 main
git checkout main
git pull origin main

# 2. 创建功能分支（命名规范：type/描述）
git checkout -b feat/add-user-registration

# 3. 开发过程中多次提交
git add .
git commit -m "feat(auth): 添加用户注册表单"
git commit -m "feat(auth): 添加邮箱验证码发送逻辑"
git commit -m "test(auth): 添加注册接口单元测试"
```

## 场景 2：日常提交规范

```bash
# ✅ 好的提交类型速查表
# feat:     新功能
# fix:      修复 bug
# docs:     文档修改
# style:    代码格式（不影响逻辑，如空格、分号）
# refactor: 重构（不是新功能也不是修 bug）
# test:     增加测试
# chore:    构建过程、辅助工具变动（如修改依赖、配置）

# 示例
git commit -m "fix(cart): 修复数量为负数时仍可下单的问题"
git commit -m "docs(readme): 更新环境配置说明"
git commit -m "refactor(api): 统一错误响应格式"
git commit -m "chore(deps): 升级 axios 到 1.6.0"
```

## 场景 3：同步 main 分支（避免冲突）

```bash
# 每天开始工作前或准备提 PR 时执行
git checkout feat/add-user-registration
git fetch origin main
git rebase origin/main  # 保持历史线性，或用 git merge

# 如果有冲突，解决后：
git add .
git rebase --continue
```

# 三、提交 Pull Request 前检查清单

```bash
# 1. 运行测试
npm test  # 或 pytest、mvn test 等

# 2. 检查提交历史是否清晰
git log --oneline -5

# 3. 压缩多余的 wip/fixup 提交（可选）
git rebase -i HEAD~3  # 合并最近 3 个提交

# 4. 推送到远程
git push origin feat/add-user-registration
```

# 四、PR 模板（直接复制到 .github/pull_request_template.md）

```markdown
## 做了什么
<!-- 简洁描述，例如：新增用户注册功能，包含表单、验证码、数据库存储 -->

## 为什么
<!-- 背景和动机，关联 issue -->
Closes #123

## 怎么测试
- [ ] 单元测试通过（`npm test` 输出绿色）
- [ ] 手动测试步骤：
  1. 访问注册页面
  2. 输入手机号，点击获取验证码
  3. 输入验证码和密码，点击注册
  4. 验证数据库新增用户记录
- [ ] Edge case：手机号已注册时提示错误

## 截图
<!-- UI 变化请贴前后对比图 -->

## 注意事项
<!-- Reviewer 需要特别关注的点，例如：修改了核心认证逻辑 -->
- 本次变更依赖 Redis 服务，请确认 staging 环境已配置
```

# 五、Review 与合并

## Reviewer 职责（30分钟内完成）

```bash
# 1. 拉到本地测试
git fetch origin pull/123/head && git checkout FETCH_HEAD
npm install && npm test

# 2. 逐行审查重点关注：
- 逻辑错误
- 安全漏洞（如 SQL 注入、XSS）
- 边界条件处理
- 测试覆盖率

# 3. 反馈方式
# - 阻塞性问题：Request Changes
# - 建议优化：Comment
# - 通过：Approve
```

## 作者合并

```bash
# 合并方式选择
# ✅ squash merge（推荐）：将整个 PR 的多个提交压缩为 1 个，main 历史干净
# ✅ rebase merge：保持线性历史，但保留所有提交
# ❌ create merge commit：会产生分叉，不推荐

# 合并后删除远程分支
git push origin --delete feat/add-user-registration
```

# 六、紧急修复流程（Hotfix）

```bash
# 1. 从 main 切出修复分支
git checkout main
git pull
git checkout -b fix/payment-timeout

# 2. 修复并提交
git commit -m "fix(payment): 修复支付宝回调超时导致订单状态不一致"

# 3. 提 PR 到 main（紧急时例外流程）
# 需要通过 CI，但可以豁免 review 时间要求

# 4. 合并后立即打 tag
git checkout main
git pull
git tag v1.2.1 -m "紧急修复支付超时问题"
git push origin v1.2.1
```

# 七、常见问题速查

## Q1：提交后想修改上次的 message
```bash
git commit --amend -m "feat(api): 新的正确描述"
```


## Q2：不小心提交到 main 怎么办

```bash
# 1. 立即在 GitHub 上禁用 push（如果有分支保护，应已阻止）
# 2. 本地创建新分支保留改动
git branch temp-branch
# 3. 回退 main
git reset --hard origin/main
```

