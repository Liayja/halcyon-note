
该Git实践参考现代Git工作流实践指南: [现代 Git 工作流实践指南](现代%20Git%20工作流实践指南.md)

# 一、仓库初始化设置

## Branch protect rules设置
在Github仓库的网页段设置分支保护，如下图：

![](../../Assets/Obsidian%20笔记在Github上的提交规范实践/file-20260513173913452.png)

**首选 "Add branch ruleset"（规则集）**。它是 GitHub 推出的新一代保护机制，功能更强大、更灵活，是未来的方向。

|对比维度|Branch Ruleset（规则集）✨ 推荐|Classic Branch Protection（经典保护）|
|---|---|---|
|**定位**|新一代，功能更强大|传统方式，功能较基础|
|**管理方式**|多个规则可以打包成一个规则集|每个分支单独配置一个规则|
|**规则数量**|一个规则集可以包含**多条规则**|每个分支只能有一套规则|
|**启用/禁用**|可以随时开关，不用删除|只能删除或保留|
|**谁可见**|有读取权限的人都能看到|只有管理员能看到|
|**适用范围**|分支、标签、通配符模式（如 `feature/*`）[](https://dev.to/piyushgaikwaad/branch-protection-rules-vs-rulesets-the-right-way-to-protect-your-git-repos-305m#comments)|主要是分支|
|**创建时校验**|✅ 分支**创建前**就能校验（如检查命名规范）[](https://dev.to/piyushgaikwaad/branch-protection-rules-vs-rulesets-the-right-way-to-protect-your-git-repos-305m#comments)|❌ 只能保护**已存在**的分支|
###    具体选项

#### 字段 1：Ruleset name（规则集名称）

> **填什么**：一个你能看懂的名字

|场景|推荐名称|
|---|---|
|保护 main 分支|`Protect Main` 或 `main-branch-rules`|
|保护所有 feature 分支|`Feature Branch Rules`|

**笔记仓库**：填 `Protect Main`

![](../../Assets/Obsidian%20笔记在Github上的提交规范实践/file-20260513215542822.png)

#### 字段 2：Enforcement status（执行状态）

> **填什么**：选 `Active`

| 选项           | 含义                   | 使用场景                    |
| ------------ | -------------------- | ----------------------- |
| **Active**   | 规则生效，违反时会被阻止         | ✅ 配置完成后选这个              |
| **Evaluate** | 只评估不阻止，在 PR 中显示警告但不拦 | 首次配置时先选这个，测试一下再切 Active |
| **Disabled** | 关闭规则                 | 临时不需要时用                 |

**建议**：首次配置可以先选 **Evaluate**，确认没问题再改为 **Active**

实测实际的设置中并没有Evaluate选项，直接选择Active即可。


#### 字段 3：Target branches（目标分支）

> **这里最关键**，指定规则应用到哪些分支

|模式|写法|效果|
|---|---|---|
|保护唯一 main|`main`|只保护 main 分支|
|保护所有 main 类|`main`、`master`|同时保护多个|
|保护所有 feature|`feature/*`|所有以 feature/ 开头的分支|
|保护所有分支|`~ALL`|所有分支都受保护|

![](../../Assets/Obsidian%20笔记在Github上的提交规范实践/file-20260513215657521.png)

| 选项                         | 作用                                          | 你的笔记仓库选哪个？ |
| -------------------------- | ------------------------------------------- | ---------- |
| **Include default branch** | 只保护默认分支（通常是 `main` 或 `master`）              | ✅ **选这个**  |
| **Include all branches**   | 保护**所有**分支（包括 `main`、`feature/*`、`fix/*` 等） | ❌ 不推荐      |
|                            |                                             |            |

#### 字段 4：Rules（核心规则）

这是规则集的核心部分，勾选你需要的规则。

|规则|作用|你的项目要不要勾|
|---|---|---|
|**Require pull request before merging**|禁止直接 push，必须通过 PR 合并|✅ **强烈推荐**|
|**Require status checks**|PR 合并前必须通过 CI 测试|⚠️ 有 CI 再勾|
|**Require conversation resolution**|PR 中的评论必须全部解决|✅ 推荐|
|**Require signed commits**|要求提交有 GPG 签名|❌ 个人项目不需要|
|**Require linear history**|禁止 merge commit，保持历史线性|⚠️ 可选，看喜好|
|**Block force pushes**|禁止强制推送（`git push --force`）|✅ **推荐**|
|**Block deletions**|禁止删除分支|✅ 推荐|

**笔记仓库推荐勾选**：

- ☑️ Require pull request before merging
    
- ☑️ Require conversation resolution
    
- ☑️ Block force pushes
    
- ☑️ Block deletions



#### 字段 5：Bypass list（豁免名单）

> **这个字段很多人会忽略，但很重要！**

哪些账号可以**绕过**这些规则？

|选项|含义|你的项目怎么填|
|---|---|---|
|**Repository admins**|仓库管理员（就是你）|可以勾上，方便紧急情况直接 push|
|特定用户/团队|指定某人可绕过|个人项目不需要|

> ⚠️ 注意：如果不勾任何人，那**你自己**也不能直接 push，必须走 PR。可以理解成一种自律约束

**推荐配置**：勾上 `Repository admins`。这样平时你会走规范流程，但万一有紧急情况（比如修个严重的错别字），你还可以直接 push。

![](../../Assets/Obsidian%20笔记在Github上的提交规范实践/file-20260513215601783.png)


## 注意事项

创建ruleset后报错：
```json
Your rulesets won't be enforced on this private repository until you move to GitHub Team organization account.
```
这个提示信息说明：**你在个人免费版账号下，想对私有仓库（Private Repository）强制推行规则集**，但 GitHub 把这个功能放在了付费套餐里。

如果笔记内容不涉及敏感信息，改成公开仓库就能免费使用规则集。


# 二、Obsidian 笔记的提交规范实践

## Conventional Commits 适配笔记场景

|类型|适用场景|示例|
|---|---|---|
|`feat`|新增笔记/文档|`feat(js): 新增闭包学习笔记`|
|`fix`|修正笔记错误|`fix(react): 更正 useState 使用示例`|
|`docs`|完善说明（README、MOC）|`docs(readme): 更新项目结构说明`|
|`refactor`|重组笔记结构|`refactor(folder): 将 JS 笔记迁移到 JS 目录`|
|`style`|格式调整（加粗、列表等）|`style(post): 统一标题层级为 H2`|
|`chore`|配置相关|`chore(git): 更新 .gitignore`|

## 实际提交示例

```bash
# ✅ 好的提交
git add "JavaScript/闭包.md"
git commit -m "feat(js): 添加闭包的 5 个实用场景示例"

git add "React/Hooks/useState.md"
git commit -m "fix(react): 修正 useState 初始化函数的语法错误"

git add "MOC/后端知识图谱.md"
git commit -m "refactor(knowledge): 将后端笔记按模块重组"

git add "Templates/"
git commit -m "feat(template): 新增 PR 模板笔记"

# ❌ 避免的提交
git commit -m "update notes"
git commit -m "修改了一些内容"
git commit -m "wip"
```


# 三、Obsidian 专属分支策略

## 1. 分支类型定义

```bash
main               # 已发布的笔记（高质量、已 review）
  ├─ feat/week-3   # 每周学习主题（eg: 学习 React 18）
  ├─ fix/typo-day  # 修正错别字
  └─ refactor/moc  # 重组 MOC（内容地图）
```

## 2. 实际工作流示例

**场景：学习"TypeScript 高级类型"**

```bash
# 1. 从 main 切出分支
git checkout main
git pull origin main
git checkout -b feat/ts-advanced-types

# 2. 在 Obsidian 中写笔记（会创建/修改 .md 文件）

# 3. 分多次提交（每完成一个小节）
git add "TypeScript/高级类型/条件类型.md"
git commit -m "feat(ts): 添加条件类型 (Conditional Types) 笔记"

git add "TypeScript/高级类型/映射类型.md"
git commit -m "feat(ts): 添加映射类型 (Mapped Types) 笔记"

git add "TypeScript/高级类型/模板字面量类型.md"
git commit -m "feat(ts): 添加模板字面量类型笔记"

# 4. 更新 MOC（内容地图）
git add "MOC/TypeScript.md"
git commit -m "docs(moc): 更新 TS 高级类型章节索引"

# 5. 推送到 GitHub
git push origin feat/ts-advanced-types
```

# 四、笔记专属 PR 模板

创建 `.github/pull_request_template.md` 在你的仓库根目录：

```markdown
## 📝 笔记变更

**主题**：<!-- 例如：TypeScript 高级类型 -->

**新增笔记**：
- [ ] `TypeScript/高级类型/条件类型.md`
- [ ] `TypeScript/高级类型/映射类型.md`

**修改笔记**：
- [ ] `MOC/TypeScript.md` - 更新索引

## 🎯 学习目标
<!-- 这些笔记想要掌握什么知识点 -->
- 理解条件类型的 `extends` 和 `infer`
- 掌握 `keyof`、`in` 在映射类型中的使用

## ✅ 自检清单
- [ ] 笔记没有明显的事实错误
- [ ] 代码示例可以直接运行（如有）
- [ ] 内部链接使用 `[[双链]]` 格式
- [ ] 标签使用正确（`#js/closure` 而非 `#js`）
- [ ] 错别字检查（建议：VS Code + Code Spell Checker）

## 🔍 需要重点 Review 的部分
<!-- 哪些内容不确定对错 -->
- 条件类型中的 `infer` 示例是否正确？
- 映射类型中 `as` 的用法是否需要补充说明？

## 📚 参考资料
- [TypeScript 官方文档 - Advanced Types](https://www.typescriptlang.org/docs/handbook/2/advanced-types.html)
- [TypeScript 进阶指南](https://example.com)

```

# 五、个人 Review 流程（笔记审查）

## Step 1: 模拟 Reviewer 审查

```bash
# 完成笔记后，自己当 reviewer
# 间隔至少 10 分钟（换个视角）

git checkout main
git fetch origin feat/ts-advanced-types
git checkout feat/ts-advanced-types

# 打开 Obsidian，用 Reading View 阅读
# 检查：
# 1. 事实准确性
# 2. 示例代码能否运行
# 3. 结构是否清晰
# 4. 双链是否正确
```

## Step 2: 在 PR 中记录发现的问题

```markdown
## Review 发现的问题（自己提给自己）

❌ **需要修改**：
1. `条件类型.md` 第 34 行：`infer R` 的示例不完整
2. `映射类型.md` 缺少 `Readonly<T>` 的实际案例

✅ **好的部分**：
- 所有代码示例都实际测试过
- 双链结构清晰

💡 **改进建议**：
- 可以在末尾添加"相关笔记"区块
```

## Step 3: 修复并推送

```bash
git checkout feat/ts-advanced-types

# 在 Obsidian 中修改
git add "TypeScript/高级类型/条件类型.md"
git commit -m "fix(ts): 完善条件类型中的 infer 示例"

git add "TypeScript/高级类型/映射类型.md"
git commit -m "feat(ts): 补充 Readonly 工具类型的实际案例"

git push origin feat/ts-advanced-types
```


# 具体操作流程示例

```bash
# 1. 回到 main 分支，同步最新
git checkout main
git pull origin main

# 2. 创建分支（根据你的操作选择）
git checkout -b feat/笔记主题     # 新增笔记
# 或
git checkout -b fix/笔记主题      # 修正错误
# 或
git checkout -b docs/笔记主题     # 更新 README

# 3. 写笔记，多次提交
git add "笔记文件.md"
git commit -m "feat(主题): 完成初稿"

git add "笔记文件.md"
git commit -m "feat(主题): 补充示例代码"

# 4. 推送分支
git push origin feat/笔记主题

# 5. 去 GitHub 创建 PR → 自己 review → 合并

# 6. 回到本地，同步 main
git checkout main
git pull origin main

# 7. 删除本地分支（保持干净）
git branch -d feat/笔记主题
```

