# 为什么用Git进行笔记知识管理

Git 不仅仅适合代码，它本质上是一个**针对纯文本文件的、时间维度的“差异记录器”**。而笔记（如果是 Markdown、TXT、LaTeX 等格式）恰好就是纯文本。

## 用 Git 管理笔记的直观好处

你提到的“直观地看到每天自己记录的知识”，Git 能做得比想象中更好：

1.  **按天查看知识生长轨迹**
```bash
git log --since="2026-05-01" --until="2026-05-08" --stat
```

这能列出这 8 天里，你每天修改/新增了哪些笔记文件，每个文件加了几行、删了几行。

2. **对比“昨天的我”和“今天的我”**
```bash
git diff 昨天下午3点的commitID 今天晚上8点的commitID
```


3. **可视化分支思想**
	- 你可以开一个 `experiment` 分支，大胆地试写一套新的知识体系（比如“用费曼技巧重写所有数学笔记”），不满意直接删掉分支，不影响主线。
    
	- 如果某天突然想回到“三周前那种笔记组织方式”，`git checkout` 瞬间即可。
## 比普通文件夹多出的“超能力”

| 需求               | 普通文件夹         | Git 笔记仓库                       |
| ---------------- | ------------- | ------------------------------ |
| 看今天新增了哪些知识点      | 手动回忆/翻文件      | `git diff 昨天的commit` 高亮显示      |
| 上周五那个写了一半的段落去哪了  | 可能被误删/覆盖      | `git log -S "写了一半的段落"` 找到当时的版本 |
| 手机/电脑/平板三端同步且不限速 | 依赖网盘/GitHub空间 | Git 仓库天然分布式，任意两端互相拉取           |
| 想分享某篇笔记的“演变过程”   | 发最终版本         | 给对方一个 GitHub 链接，能看到你的心路历程      |


# Obsidian如何关联Github仓库

推荐**先在本地建立 Obsidian 笔记仓库，然后在其根目录下 `git init`，再关联到 GitHub 上的空仓库**。

这样做最直接、最自然，不会产生多余的目录层级，也更贴合 Obsidian 的使用习惯。

## 具体操作步骤（非常简洁）

1. **在 GitHub 上新建一个仓库**
    
    - 仓库名自定，**不要勾选** “Add a README file”、“Add .gitignore”、“Add a license” 这三项，保持完全空白。
        
    - 创建后，你会看到一个页面，上面显示了…“…or push an existing repository from the command line”。
        
2. **在本地打开你的 Obsidian 笔记仓库文件夹**
    
    - 就是平时用 Obsidian 打开的那个文件夹（假设路径是 `~/Documents/MyNotes`）。
        
3. **在该文件夹中打开终端，执行以下命令**

```bash
cd ~/Documents/MyNotes   # 进入你的 Obsidian 仓库目录
git init                 # 初始化本地 Git 仓库
git remote add origin https://github.com/你的用户名/仓库名.git   # 关联远程仓库
```

4. **配置 `.gitignore`**（把之前讨论的规则写进去），然后提交并推送：
```bash
# 创建 .gitignore 文件（可以用 echo 或手动编辑）
echo ".obsidian/workspace.json" >> .gitignore
echo ".obsidian/workspace-mobile.json" >> .gitignore
# ... 其他忽略规则

git add .
git commit -m "初始提交：全部现有笔记"
git push -u origin main
```

## 为什么不建议“先在 GitHub 创建并 clone 到本地”？

因为 `git clone` 会在本地生成一个**外层文件夹**（即仓库根目录）。如果你把 Obsidian 仓库直接建在这个外层文件夹内部，会导致：

- 目录结构变成 `外层仓库文件夹/笔记文件`，而不是笔记文件夹本身就是 Git 仓库根目录。
    
- 虽然不影响功能，但 Obsidian 的“打开仓库”会指向外层文件夹里的内容，感觉多了一层，操作上不如直接 `git init` 干净。
    

如果你希望仓库根目录就是 Obsidian 仓库的根目录，那就直接用 `git init` 的方式。

## 如何查看关联成功

```bash
git remote -v
```

- **如果关联成功**，你会看到类似这样的输出（以 `origin` 为例）：

```bash
origin  https://github.com/用户名/仓库名.git (fetch)
origin  https://github.com/用户名/仓库名.git (push)
```

- **如果没有关联**，则不会有任何输出（或显示空行）。

## 核心概念：工作区 → 暂存区 → 仓库 → 远程仓库

```text
你的笔记文件夹（工作区）
       ↓
    git add（stage all）—— 放到“暂存区”
       ↓
    git commit —— 永久保存到“本地仓库”
       ↓
    git push —— 同步到“GitHub（远程仓库）”

另外：
    git pull —— 从 GitHub 拉取最新内容到本地
    commit and sync —— 一次性完成 commit + push + pull
```


# 需要注意的点

## 哪些文件需要被忽略

### 需要同步的文件 vs 需要忽略的文件

根据 Obsidian Git 社区的最佳实践，完整的 `.gitignore` 应该是这样的：

```gitignore
# 忽略设备相关的状态文件（不同设备会不同）
.obsidian/workspace.json
.obsidian/workspace-mobile.json

# 忽略缓存和临时文件
.obsidian/cache/
.trash/
.DS_Store
Thumbs.db

# 可选：忽略特定插件的本地数据库（如果体积很大）
.smart-connections/
.pro/

# 以下是需要同步的，注意不要写进 .gitignore
# .obsidian/app.json
# .obsidian/appearance.json
# .obsidian/community-plugins.json
# .obsidian/core-plugins.json
# .obsidian/graph.json
# .obsidian/plugins/ */data.json  (按照我们之前讨论的规则)
```

## GitHub 默认分支是 `main`，而本地仓库是 `master`

现在 GitHub 新建仓库的默认分支名是 `main`，但本地是 `master`。如果想统一成 `main`：
```bash
# 方法一：在 GitHub 上把默认分支改成 master（不推荐）
# 方法二：把本地分支重命名为 main
git branch -m master main
git push -u origin main
```

# 图片如何进行Git管理

 **Git LFS** 是一个让 Git 能高效处理图片这类大文件（如数百 KB 甚至更大）的官方扩展。它通过“按需下载”来避免仓库体积变大。
 
 简单来说，Git LFS 是一个让你在继续享受 Git 版本控制的同时，又能避免仓库臃肿的官方扩展。

- **它的工作原理**：Git LFS 会将你指定的大文件（图片、视频、数据集等）替换成一个只有几百字节的“文本快捷方式”并替你管理。真正的图片文件本体，则被存储到独立的 Git LFS 服务器上。
    
- **它如何让 Git 变快**：当你或其他人 `git clone` 或 `git checkout` 仓库时，默认只会下载这些小巧的“快捷方式”和你的文字笔记。只有当需要查看具体某张图片时（比如用软件打开它），Git LFS 才会从服务器上把真正的图片文件下载下来。这就让日常的 Git 操作，感觉上会快很多。

只需几步，你就可以在自己的笔记仓库中配置好 Git LFS：

### 🛠 第一步：在本地安装 Git LFS

要使用Git LFS，你需要在你的电脑上安装它。这里是针对你的操作系统（Windows）的步骤：

1. **下载安装包**：访问Git LFS官网 [git-lfs.com](https://git-lfs.com/) 下载。
    
2. **安装**：运行下载的安装程序（例如 `git-lfs-windows-v3.6.0.exe`），按照提示完成安装。
    
3. **全局初始化**：安装完成后，打开命令行工具（CMD 或 Git Bash），输入以下命令并运行：
```bash
git lfs install
```
这一步是为Git全局环境设置好必要的配置，在电脑上执行一次即可

### 🔗 第二步：将 Obsidian 与 Git LFS 集成

在你的本地 Git LFS 准备好之后，就可以在Obsidian里使用了。

- **常规配置流程**：在你的Obsidian笔记库中配置好 `Obsidian Git` 插件，并在笔记库的根目录通过命令行使用 `git lfs track "*.png"` 命令来指定需要进行LFS管理的文件类型[](https://www.utgd.net/article/20870)。之后，正常通过 `Obsidian Git` 插件的按钮进行提交和推送即可。

### 🔗 第三步：针对图片文件启用 LFS 追踪

1. 进入 Obsidian 笔记仓库目录:

```bash
cd ~/Documents/MyObsidianNotes
```

2. 告诉 Git LFS 要追踪哪些图片格式，运行以下命令（可以一次追踪多种格式）：
```bash
git lfs track "*.png" "*.jpg" "*.jpeg" "*.gif" "*.webp" "*.bmp"
```

如果还有其他图片格式（如 `.svg`、`.ico`），也可以一并加上。
**这条命令做了什么？**  
它会创建一个名为 `.gitattributes` 的文件（如果不存在的话），并在其中写入类似下面的内容：
```bash
*.png filter=lfs diff=lfs merge=lfs -text
*.jpg filter=lfs diff=lfs merge=lfs -text
*.jpeg filter=lfs diff=lfs merge=lfs -text
...
```

这个文件告诉 Git：以后所有符合这些后缀名的文件，都用 LFS 来处理。

3. 提交 `.gitattributes` 到仓库
为了让 LFS 配置生效，需要把 `.gitattributes` 文件提交进去：
```bash
git add .gitattributes
git commit -m "启用 Git LFS 追踪图片文件"
```

# 使用Custom Attachment Location集中管理附件

## 方案对比：总目录 vs 分散目录

|维度|方案一：总目录（根目录下一个 `attachments` 文件夹，内部按笔记名/日期建子目录）|方案二：分散目录（每个笔记文件夹下自带一个 `media` 或 `attachments` 子文件夹）|
|---|---|---|
|**附件集中程度**|✅ 所有附件都在一个根目录下，仓库结构清晰，一眼看到所有图片资源。|❌ 附件分散在各处，随着笔记增多，附件目录会遍布整个仓库。|
|**Git LFS 配置**|✅ 只需一条规则 `attachments/**` 即可追踪全部附件，简单且不易遗漏。|⚠️ 需要配置 `**/media/**` 或类似通配符，也能实现，但规则略复杂。|
|**移动/重命名笔记**|❌ 移动笔记文件后，其对应的附件子目录不会自动跟随，需要手动迁移或脚本处理。|✅ 移动整个笔记文件夹时，附件文件夹自动跟随，路径相对关系不变。|
|**Obsidian 内部链接**|链接形式：`![[attachments/笔记A/图片.png]]`，稍长但可接受。|链接形式：`![[media/图片.png]]`，更短。|
|**清理孤立附件**|容易集中扫描，找出没有被任何笔记引用的附件。|分散在各处，清理孤立附件需要遍历全仓库。|
|**备份/迁移**|只需备份一个 `attachments` 目录。|需要备份所有分散的 `media` 目录。|
## 具体做法

1. **在仓库根目录创建一个 `attachments` 文件夹**。
    
2. **使用 `Custom Attachment Location` 插件，设置路径模板为**：
```text
attachments/${noteFileName}
```
这样当你编辑 `笔记A.md` 时，粘贴的图片会自动保存到 `attachments/笔记A/图片.png`；编辑 `笔记B.md` 时，图片保存到 `attachments/笔记B/`。

3. **Git LFS 配置**：
```bash
git lfs track "attachments/**"
```

一条规则管理所有附件。

![[file-20260509162320691.png]]
