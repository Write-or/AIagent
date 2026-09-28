学习git
# Git 使用教程

我以你当前这个仓库 `C:\Users\Administrator\Desktop\AIagent` 为实例讲，你边看边对照实际状态。

---

## 一、核心概念：三个区域

理解 Git 只需要记住三个区域和它们之间的搬运方向：

```
工作区              暂存区              本地仓库            远程仓库
(Working Dir)      (Staging/Index)    (Repository)       (Remote)
    │                   │                  │                  │
    │   git add         │   git commit     │   git push       │
    ├──────────────────>├─────────────────>├─────────────────>│
    │                   │                  │                  │
    │<──────────────────┼──────────────────┼──────────────────┤
    │   git restore     │   git reset      │   git pull/fetch │
```

- **工作区**：你眼睛看到的文件夹，直接改文件就是改这里。
- **暂存区**：一个"准备提交清单"。`git add` 就是把改动放进清单。
- **本地仓库**：`.git` 目录，存所有历史快照。`git commit` 把暂存区打包成一个快照存进来。
- **远程仓库**：GitHub/Gitee 上的副本。`push` 上传，`pull` 下载。

**关键理解**：`add` 和 `commit` 是两步，不是一步。这是 Git 最反直觉的地方，也是它比"直接保存版本"强大的原因——你可以只提交一部分改动。

---

## 二、日常最常用的 6 条命令

### 1. `git status` —— 看当前状态（最高频）

```powershell
git status
```

你现在的输出：

```
On branch main
No commits yet

Changes to be committed:          ← 暂存区里的（已 add，待提交）
	new file:   AI-Agent-学习规划.md
	new file:   node/day-1.md

Changes not staged for commit:    ← 工作区改了但没 add 的
	modified:   node/day-1.md
```

翻译成人话：`node/day-1.md` 已经 add 过一次了，但**之后你又改了它**，新改动还没 add。所以它同时出现在两个列表里。

**养成习惯：动手前先 `git status`，永远先看状态。**

### 2. `git add` —— 加入暂存区

```powershell
git add node/day-1.md        # 加单个文件
git add node/                # 加整个目录
git add .                    # 加当前目录下所有改动
git add -A                   # 加全仓库所有改动（含删除）
```

### 3. `git commit` —— 提交快照

```powershell
git commit -m "提交说明"
git commit -m "标题" -m "详细描述"     # 两段式：标题+正文
git add . ; git commit -m "xxx"       # 常见连写
```

**提交说明怎么写**：说清"做了什么、为什么"，不是"改了点东西"。好的例子：

```
fix: 修复登录页手机号校验失效
feat: 新增用户导出 CSV 功能
docs: 补充 day-1 学习笔记
```

### 4. `git log` —— 看历史

```powershell
git log                      # 完整历史
git log --oneline            # 一行一条，最常用
git log --oneline --graph --all    # 带分支图形
git log -3                   # 最近 3 条
git log -p node/day-1.md     # 看某文件的每次改动内容
```

### 5. `git diff` —— 看改动内容

```powershell
git diff                     # 工作区 vs 暂存区（还没 add 的改动）
git diff --staged            # 暂存区 vs 上次提交（即将提交的内容）
git diff HEAD                # 工作区 vs 上次提交（所有未提交改动）
```

**提交前用 `git diff --staged` 确认一遍自己到底要提交什么**，这是防手滑的关键一步。

### 6. `git restore` —— 撤销（危险，见第四节）

---

## 三、分支：Git 真正的威力

分支就是"从某个点分叉出去，独立开发，不影响主线"。

```powershell
git branch                   # 列出本地分支（* 标记当前分支）
git branch -a                # 含远程分支
git switch main              # 切换到 main 分支
git switch -c feature/login  # 新建并切换到 feature/login
git switch -c fix/bug-123    # 分支名用 / 分层，是惯例
```

合并：

```powershell
git switch main              # 先回到主干
git merge feature/login      # 把 feature/login 合进来
```

**新手推荐工作流**：

```
1. git switch -c feature/xxx     # 从 main 拉新分支
2. 改代码 → git add . → git commit -m "..."    # 反复几次
3. git switch main
4. git merge feature/xxx         # 合并
5. git branch -d feature/xxx     # 删掉已合并的分支
```

好处：main 永远是能跑的版本，试验性的东西坏了直接删分支就行。

---

## 四、撤销：出错怎么办

这是最容易慌的地方，按"改动在哪个区域"分四种情况：

| 我想撤销… | 命令 | 是否丢数据 |
|---|---|---|
| 工作区改动（还没 add） | `git restore <文件>` | **丢**，谨慎 |
| 已 add 但没 commit | `git restore --staged <文件>` | 不丢，只退出暂存区 |
| 刚 commit 但没 push | `git commit --amend` | 不丢，修改这次提交 |
| 已 push 到远程 | `git revert <commit>` | 不丢，生成反向提交 |

具体展开：

```powershell
# 情况1：文件改乱了，想恢复成上次提交的样子（丢弃本地修改！）
git restore node/day-1.md

# 情况2：add 错了，想从暂存区拿出来（改动还在工作区）
git restore --staged node/day-1.md

# 情况3：刚提交完发现说明写错/漏了文件
git add 漏掉的文件
git commit --amend            # 重新编辑提交说明

# 情况4：已经 push 了，想撤销某次提交
git revert <commit-hash>      # 生成一个"反向操作"的新提交
```

**核心原则**：已经 push 出去的提交，永远不要用 `reset --hard` 去改历史，用 `revert`。

⚠️ **`git reset --hard` 会永久丢弃改动**，除非你确定，否则别用。

---

## 五、连接远程仓库

### 关联远程

```powershell
git remote add origin https://github.com/用户名/仓库名.git
git remote -v                 # 查看已关联的远程
```

### 首次推送

```powershell
git branch -M main            # 确保主分支叫 main
git push -u origin main       # -u 记住关联，以后直接 git push
```

### 日常同步

```powershell
git push                      # 上传本地提交
git pull                      # 下载远程更新并合并
git fetch                     # 只下载不合并（更安全，可先看看）
```

### 你的情况

你现在的仓库还没有远程：

```powershell
git remote -v      # 空输出
```

想推 GitHub 的话，先在 GitHub 建空仓库（**不要**勾选 README 初始化），然后：

```powershell
git remote add origin https://github.com/你的用户名/AIagent.git
git push -u origin main
```

推送时会弹认证窗口，国内网络可能要配代理或改用 SSH。

---

## 六、你现在这个仓库该怎么走

当前状态：两个文件在暂存区，`node/day-1.md` 还有未暂存的改动。

```powershell
# 1. 先看看 day-1.md 又改了什么
git diff

# 2. 确认没问题就把新改动也加进来
git add node/day-1.md

# 3. 提交（你的第一个 commit！）
git commit -m "docs: 添加 AI Agent 学习规划和 day-1 笔记"

# 4. 确认成功
git log --oneline
```

---

## 七、必须知道的几个坑

### 1. `.gitignore` —— 别把垃圾提交进去

在仓库根目录建 `.gitignore`：

```gitignore
# 依赖
node_modules/
__pycache__/
*.pyc

# 构建产物
dist/
build/
*.exe

# 环境变量（含密钥，绝对不能提交）
.env
.env.local

# 编辑器
.vscode/
.idea/

# 系统文件
Thumbs.db
Desktop.ini
```

### 2. 换行符问题（Windows 特有）

Windows 用 `CRLF`，Linux/Mac 用 `LF`。你的系统配置里 `core.autocrlf=true` 已自动处理，一般不用管。但跨平台协作时如果出现"整个文件都改了"的诡异 diff，就是这个原因。

### 3. 中文文件名乱码

你的仓库文件名是中文，配置 `core.quotepath false` 后 `git status` 会正常显示中文而不是 `\346\226\207` 这种转义。这个我已经帮你配好了。

### 4. 提交信息写错了已经 push

不要慌，也不要 `--force` 硬推。用 `git revert` 生成新提交来纠正。

---

## 八、命令速查表

```powershell
# 状态与查看
git status                    # 当前状态
git log --oneline --graph     # 历史
git diff                      # 未暂存改动
git diff --staged             # 已暂存改动

# 提交流程
git add <文件>                # 暂存
git commit -m "说明"          # 提交
git push                      # 推送

# 分支
git switch -c <分支名>        # 新建并切换
git switch <分支名>           # 切换
git merge <分支名>            # 合并
git branch -d <分支名>        # 删除

# 撤销
git restore <文件>            # 丢弃工作区改动 ⚠️
git restore --staged <文件>   # 移出暂存区
git revert <hash>             # 反向提交
git reset --hard <hash>       # 硬回退 ⚠️⚠️ 慎用

# 远程
git remote add origin <url>   # 关联远程
git pull                      # 拉取
git push                      # 推送
```

---

## 建议的学习路径

1. **今天**：把上面第六节的三条命令跑一遍，完成你的第一个 commit
2. **本周**：养成 `status → add → diff --staged → commit` 的肌肉记忆
3. **然后**：练分支，自己建个分支改点东西再合并
4. **最后**：推 GitHub，学 PR（Pull Request）流程

需要我帮你做第一次提交，或者配好 GitHub 远程仓库吗？