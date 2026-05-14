
# 为什么不能直接push到main?

##  想象一个场景

你在写一篇重要的文章，只有**最终版本**应该发布出去。

```text
❌ 错误做法（你现在的方式）：
草稿1 → 直接发布
草稿2 → 直接发布  
草稿3 → 直接发布

问题：读者会看到你写到一半的垃圾内容
```

```text
✅ 正确做法：
草稿1 → 私密草稿箱
草稿2 → 私密草稿箱
草稿3 → 私密草稿箱
完成 → 审核 → 发布

好处：只有成品才被看到
```

**main 分支 = 已发布的作品集**，应该永远保持高质量、可用的状态。

## 分支是什么

### 用文件夹类比

```bash
# 你现在的情况（只有一个 main 分支）
main 文件夹/
  ├─ 你的所有文件（混在一起）

# 有分支的情况
main 文件夹/           # 只有正式内容
feat/login 文件夹/     # 单独的空间，写登录功能
feat/pay 文件夹/       # 单独的空间，写支付功能
fix/bug 文件夹/        # 单独的空间，修 bug
```

**关键点**：

- 不同分支之间**互相隔离**，不会干扰
- 可以在不同分支之间**切换**
- 写完后可以**合并**到一起

### 演示
```bash
# 1. 查看当前所有分支（* 表示当前所在分支）
git branch
* main

# 2. 创建新分支（就像是复制了一份完整的代码）
git branch feat-hello

# 3. 切换到新分支
git checkout feat-hello
# 或合并写法（创建并切换）
git checkout -b feat-hello

# 4. 现在你在分支里，修改文件不会影响 main
echo "新功能" >> README.md
git add .
git commit -m "feat: 添加新功能"

# 5. 切换回 main，你会发现 README.md 没变！
git checkout main
cat README.md  # 看不到刚才的修改
```


##  场景：给笔记添加一个"读书笔记"板块

```bash
# === 第一步：确保 main 是最新的 ===
git checkout main
git pull origin main

# === 第二步：创建新分支 ===
git checkout -b feat/reading-notes
# 分支名解释：feat=新功能，reading-notes=具体内容

# === 第三步：在分支上正常写内容 ===
# （在 Obsidian 中写笔记...）

git add "读书笔记/原子习惯.md"
git commit -m "feat(book): 添加《原子习惯》读书笔记"

git add "读书笔记/深度工作.md"
git commit -m "feat(book): 添加《深度工作》读书笔记"

git add "README.md"
git commit -m "docs: 更新目录，添加读书笔记板块"

# === 第四步：推送到 GitHub ===
git push origin feat/reading-notes
# 现在 GitHub 上有了一个新分支

# === 第五步：创建 PR（Pull Request）===
# 去 GitHub 网页操作（下面详细说）
```



#  Pull Request (PR) 是什么？

## 通俗解释

**PR = 你发出一个请求："请把我的修改合并到 main 吧"**

就像是你写完文章后，提交给主编审核：

1. 你提交申请
    
2. 主编看看内容
    
3. 没问题就批准发布

## 实际操作（GitHub 网页）

```bash
# 你 push 分支后，GitHub 会显示黄色提示框：
# "feat/reading-notes had recent pushes"
# 点击旁边的 "Compare & pull request" 按钮
```
然后你会看到一个表单：

```markdown
# 标题自动填充了你的最后 commit
feat(book): 添加《深度工作》读书笔记

# 描述区域（需要你填写）
## 做了什么
- 新增读书笔记板块
- 添加《原子习惯》《深度工作》两篇笔记

## 为什么
建立个人知识管理系统，记录读书思考

## 怎么测试
- [x] 本地打开 Obsidian 验证链接正确
- [x] 检查没有错别字

# 右侧可以选择：
- Reviewers: （谁审查，个人项目可以不选）
- Labels: （标签，如 enhancement）
```

点击 **Create pull request** 按钮 → PR 创建成功！


# 合并`Merge`是什么？

## 三种合并方式

|方式|说明|效果|推荐度|
|---|---|---|---|
|**Create merge commit**|保留所有历史|main 上有分叉|❌ 不推荐|
|**Squash and merge**|把多个commit压成一个|main 历史干净|✅ 个人强烈推荐|
|**Rebase and merge**|保持线性历史|保留每个commit|✅ 团队常用|

## 实际选择（个人项目）

```bash
# 在 PR 页面，点击 Merge 按钮旁边的下拉箭头
# 选择 "Squash and merge"

# 合并后：
# - 你的 3 个 commit 会变成 1 个 commit
# - commit 信息可以编辑成：
#   "feat(book): 新增读书笔记板块（原子习惯、深度工作）"
```



# `git push`的三种写法区别

|命令|核心区别|适用场景|
|---|---|---|
|`git push`|最简写法，依赖**已有配置**|第二次及以后推送当前分支|
|`git push origin main`|完整写法：推送到 origin 仓库的 main 分支|任何时候都可用，明确指定|
|`git push -u origin main`|完整写法 + **建立追踪关系**|**第一次**推送新分支时使用|

## 1. `git push origin main` - 最标准的写法

```bash
# 意思：把当前本地的 main 分支，推送到远程 origin 仓库的 main 分支
git push origin main
```
**拆解：**

- `origin`：远程仓库的默认名字（相当于一个快捷方式，指向你的 GitHub 仓库地址）
- `main`：本地分支名

**使用场景：**

- 任何时候都可用，明确告诉 Git 推送到哪里
- 推荐新手使用，不容易出错

## 2. `git push -u origin main` - 第一次推送新分支

```bash
# -u 是 --set-upstream 的缩写
git push -u origin main
```

**作用：** 推送 + **建立追踪关系**

**什么是追踪关系？**

- 告诉 Git："以后这个本地 main 分支，默认就推送到 origin/main"
- 建立后，下次可以简写为 `git push`

**实际演示：**
```bash
# 第一次推送新分支（必须完整写）
git push -u origin feature/hello

# 第二次及以后推送同一个分支（可以简化）
git push  # 会自动推送到 origin/feature/hello

```

## 3. `git push` - 最简写法

```bash
git push
```

**前提条件：**

- 当前分支已经有**上游追踪关系**（之前用过 `-u`）
- 或者 Git 配置了默认行为

**等价于：**

- 如果当前分支是 main，且之前用过 `git push -u origin main`
- 那么 `git push` = `git push origin main`

**问题：**

- 如果当前分支没有追踪关系，Git 会报错：
```json
fatal: The current branch feature/hello has no upstream branch.
```




