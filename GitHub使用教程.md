# GitHub 使用教程（Windows · 从零到第一次推送）

> 本机环境：Windows + Git 2.56（`D:\Git`）
> 本地仓库：`D:\宁致远\git-repo`（已初始化，分支 `main`，已有首个提交）
> 账号信息：`宁致远 <13581661445@163.com>`

---

## 目录

1. [先弄清三个概念](#1-先弄清三个概念)
2. [注册 GitHub 账号](#2-注册-github-账号)
3. [网络问题：先解决能不能连上](#3-网络问题先解决能不能连上)
4. [认证方式：SSH 密钥（本机已配好）](#4-认证方式ssh-密钥本机已配好)
5. [在 GitHub 上创建空仓库](#5-在-github-上创建空仓库)
6. [本地仓库关联远程并首次推送](#6-本地仓库关联远程并首次推送)
7. [日常开发循环（90% 的时间只用这 5 条）](#7-日常开发循环90-的时间只用这-5-条)
8. [分支与 Pull Request](#8-分支与-pull-request)
9. [把别人的项目拿到本地](#9-把别人的项目拿到本地)
10. [后悔药：撤销修改与提交](#10-后悔药撤销修改与提交)
11. [常见报错速查表](#11-常见报错速查表)
12. [本机现状备忘](#12-本机现状备忘)
13. [接下来 30 分钟清单](#13-接下来-30-分钟清单)
14. [进阶方向](#14-进阶方向)

---

## 1. 先弄清三个概念

| 名词 | 是什么 | 打个比方 |
|---|---|---|
| **Git** | 本地版本管理工具，记录你每次改动的快照 | 你电脑上的"时光机" |
| **GitHub** | 托管 Git 仓库的网站，同时提供协作功能 | 存时光机录像带的"云盘 + 社交平台" |
| **仓库（Repository / repo）** | 一个被 Git 管理的文件夹 | 一盘录像带 |

关系：`Git` 负责在**本地**记录改动；`GitHub` 负责把仓库**存到云端**并和别人协作。
本地有提交、云端没同步，别人就看不到；云端有更新、本地没拉取，你就会落后。

**四个区域**（理解这个，后面的命令就都能对上号）：

```
工作区（你编辑的文件）
   │ git add
   ▼
暂存区（staging，准备提交的内容）
   │ git commit
   ▼
本地仓库（.git 目录，历史提交）
   │ git push
   ▼
远程仓库（GitHub）
```

---

## 2. 注册 GitHub 账号

1. 浏览器打开 <https://github.com/signup>
2. 依次填写：
   - **Email**：`13581661445@163.com`
   - **Password**：至少 15 位，或至少 8 位且包含数字和小写字母（GitHub 的规则）
   - **Username**：**慎重选择**，它会出现在所有仓库地址里，例如 `https://github.com/你的用户名/git-repo`，以后一般不改。建议用英文小写 + 数字，如 `ningzhiyuan`。
3. 完成人机验证，查收邮件点验证链接。
4. 注册后建议做两件事：
   - 右上角头像 → **Settings** → 左侧 **Public profile**：填 Name（可填中文"宁致远"）、头像
   - **Settings** → **Password and authentication** → 开启 **Two-factor authentication（2FA）**，用手机 App（如 Microsoft Authenticator）扫码，并**保存好恢复码**

> 用户名一旦选好，本地推送地址就固定为 `git@github.com:用户名/仓库名.git`，请记下来。

---

## 3. 网络问题：先解决能不能连上

我在本机实测了 GitHub 各域名的连通性（2026-10-19）：

| 地址 | 结果 | 说明 |
|---|---|---|
| `https://github.com`（网页、HTTPS 推送） | ❌ 连接被重置 | 网页打不开、HTTPS 推送会失败 |
| `https://raw.githubusercontent.com` | ❌ 超时 | 网页看不了代码 |
| `https://api.github.com` | ✅ 200 | 部分 API 可用 |
| `https://codeload.github.com` | ✅ 200 | 下载压缩包可用 |
| `ssh.github.com:443` | ✅ 可建立 SSH 连接 | **这是能用的一条路** |

结论：**网页注册/浏览需要先解决访问问题，代码推送走 SSH 443 端口则已经可用**（本机我已经配好，见下一节）。

解决网页访问的常见办法（任选其一，自行判断风险）：

1. **Watt Toolkit（原 Steam++）**：开源工具，自带 GitHub 加速（修改 hosts），国内常用。下载后勾选"GitHub"，点"一键加速"。
2. **代理软件**：如果你已有代理（Clash 等），开启 TUN/系统代理即可；Git 走代理时再执行：
   ```powershell
   git config --global http.proxy http://127.0.0.1:7890
   git config --global https.proxy http://127.0.0.1:7890
   # 取消代理
   git config --global --unset http.proxy
   ```
3. **镜像站**：如 `https://bgithub.xyz`（把 github.com 换成它可看代码），但注册账号、提交 PR 等操作仍需官网，镜像只适合临时查看。

> 没有账号、网页又打不开时，先处理加速工具，这是后面所有步骤的前提。

---

## 4. 认证方式：SSH 密钥（本机已配好）

GitHub 从 2021 年起**不再接受账号密码推送**，必须用下面两种方式之一：

| 方式 | 特点 | 适合 |
|---|---|---|
| HTTPS + 个人访问令牌（PAT） | 每次要管令牌，浏览器弹窗登录 | 临时用、公司电脑 |
| **SSH 密钥** | 一次配置，永久免密 | 长期使用（推荐） |

### 本机已完成的部分

- 已生成密钥：`C:\Users\nssd\.ssh\id_ed25519`（私钥，绝不外传）和 `id_ed25519.pub`（公钥，用来给 GitHub）
- 已写入 `C:\Users\nssd\.ssh\config`，把 `github.com` 的 22 端口绕到 **443 端口**（国内 22 端口常被墙）：

  ```
  Host github.com
      HostName ssh.github.com
      Port 443
      User git
      IdentityFile C:/Users/nssd/.ssh/id_ed25519
  ```

- 已实测：`ssh -T git@github.com` 能连到 GitHub，返回 `Permission denied (publickey)` —— 这只说明**公钥还没加到账号里**，属正常。

### 你需要做的一步：把公钥添加到 GitHub

1. 复制公钥（在 PowerShell 里执行，已经在剪贴板里了）：
   ```powershell
   Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub" | Set-Clipboard
   ```
   公钥内容：
   ```
   ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAPUUY0mIzcVGa57Iv4OL+E8E8as72yXRuCXq9wEz9Hh 13581661445@163.com
   ```
2. 浏览器打开 <https://github.com/settings/keys>
3. 点 **New SSH key**
   - **Title**：随便写，如 `我的电脑-Windows`
   - **Key type**：`Authentication Key`
   - **Key**：粘贴（`Ctrl+V`）
4. 点 **Add SSH key**
5. 回到 PowerShell 验证：
   ```powershell
   ssh -T git@github.com
   ```
   看到下面这句就成功了（`你的用户名` 会显示你自己的）：
   ```
   Hi 你的用户名! You've successfully authenticated, but GitHub does not provide shell access.
   ```

> 如果以后要换电脑或重装系统，重复"生成密钥 → 上传公钥"即可。私钥泄露要立刻在设置页删掉对应公钥并重新生成。

---

## 5. 在 GitHub 上创建空仓库

1. 右上角 **+** → **New repository**（或直接打开 <https://github.com/new>）
2. 填写：
   - **Repository name**：`git-repo`（和你本地文件夹同名，便于对应）
   - **Description**：可选，如"我的学习/项目仓库"
   - **Public / Private**：私有要谨慎（免费也有隐私额度），初学建议 **Public** 或 **Private** 都行
3. **三个初始化选项全部不要勾**：
   - ❌ Add a README file
   - ❌ Add .gitignore
   - ❌ Choose a license

   原因：本地仓库已经有提交，云端再初始化会出现"两段互不相干的历史"，推送会报错。**这是新手第一大坑。**
4. 点 **Create repository**。页面会显示一段"快速设置"命令，先别管它，用下面的步骤。

---

## 6. 本地仓库关联远程并首次推送

在 PowerShell 里执行（把 `你的用户名` 换成实际的）：

```powershell
cd D:\宁致远\git-repo

# 1. 关联远程仓库，命名为 origin
git remote add origin git@github.com:你的用户名/git-repo.git

# 2. 检查关联是否正确
git remote -v
# 预期输出两行：
# origin  git@github.com:你的用户名/git-repo.git (fetch)
# origin  git@github.com:你的用户名/git-repo.git (push)

# 3. 确认分支名是 main
git branch --show-current

# 4. 首次推送，并把本地 main 与远程 main 绑定
git push -u origin main
```

成功后：

- 终端显示 `branch 'main' set up to track 'origin/main'`
- 刷新 GitHub 仓库页面，能看到 `README.md`、`.gitignore` 和提交记录
- 以后只需 `git push`、`git pull`，不用再写 `-u origin main`

> 地址写错了？改：
> ```powershell
> git remote set-url origin git@github.com:你的用户名/git-repo.git
> ```

---

## 7. 日常开发循环（90% 的时间只用这 5 条）

```powershell
git status                # ① 看现在有什么改动
git add .                 # ② 把改动放进暂存区（. 表示全部；也可 git add 文件名）
git commit -m "说明"       # ③ 提交到本地仓库
git push                  # ④ 推送到 GitHub
git pull --rebase         # ⑤ 拉取远程更新（开新工作前先拉）
```

**提交说明怎么写**（推荐 Conventional Commits 前缀，团队协作通用）：

| 前缀 | 含义 | 示例 |
|---|---|---|
| `feat:` | 新功能 | `feat: 增加登录页面` |
| `fix:` | 修 bug | `fix: 修复中文路径读取失败` |
| `docs:` | 文档 | `docs: 补充使用说明` |
| `refactor:` | 重构（不改行为） | `refactor: 拆分工具函数` |
| `chore:` | 杂项（依赖、配置） | `chore: 升级依赖版本` |

**几条习惯**：

- 每完成一个小功能就提交一次，别攒一星期再提交
- `git status` 随时看，`git diff` 看具体改了什么（暂存前的差异）
- `git log --oneline --graph` 看历史
- 提交前确认没有把密码、密钥、`node_modules` 提交进去（`.gitignore` 已经挡掉常见项）

---

## 8. 分支与 Pull Request

分支让你"在不影响主线的位置改东西"，改完再合并。单人项目也建议用，养成习惯。

### 8.1 开分支、改、推、合并

```powershell
# 确保主线最新
git switch main
git pull

# 从 main 开一个新分支（名字描述你在做什么）
git switch -c feature/login

# ……改代码……

git add .
git commit -m "feat: 增加登录功能"
git push -u origin feature/login
```

推送成功后，GitHub 仓库页会出现黄色提示条 **Compare & pull request**：

1. 点它 → 填标题和说明 → **Create pull request**
2. 自己检查 Files changed → 点 **Merge pull request** → **Confirm merge**
3. 合并后点 **Delete branch** 删除远程分支

本地收尾：

```powershell
git switch main
git pull
git branch -d feature/login                    # 删本地分支
git push origin --delete feature/login         # 删远程分支（网页删过就跳过）
```

### 8.2 PR 是什么、为什么值得用

PR（Pull Request）= "请求把我的分支合并进 main"。它带来：

- **差异可视化**：逐行看改了啥，比直接改 main 安全
- **讨论区**：可以评论某一行代码
- **自动化检查**：可挂 CI（GitHub Actions）跑测试
- **可追溯**：以后能查"这个功能是哪次 PR、谁合进来的"

即使一个人开发，也建议走"分支 → PR → 合并"，而不是直接往 main 推。

---

## 9. 把别人的项目拿到本地

### 9.1 只是使用/学习

```powershell
git clone git@github.com:用户名/仓库名.git          # SSH 方式（本机可用）
git clone https://github.com/用户名/仓库名.git      # HTTPS 方式（本机当前网络不通）
git clone --depth 1 <地址>                          # 只拿最新一次提交，快
```

### 9.2 想参与开源（Fork 流程）

你没有对方仓库的写权限，所以先 **Fork** 一份到自己名下：

1. 在对方仓库页点右上角 **Fork** → 创建自己的副本
2. 克隆**自己的副本**：`git clone git@github.com:你的用户名/项目名.git`
3. 加一个"上游"指向原仓库，方便同步：
   ```powershell
   cd 项目名
   git remote add upstream git@github.com:原作者/项目名.git
   git remote -v
   ```
4. 开分支改代码、提交、推到自己 fork：
   ```powershell
   git switch -c fix/某问题
   git add . && git commit -m "fix: 修正xxx"
   git push -u origin fix/某问题
   ```
5. 回到 GitHub 你的仓库页，点 **Contribute → Open pull request**，向原仓库发起 PR
6. 定期同步上游更新：
   ```powershell
   git fetch upstream
   git switch main
   git merge upstream/main
   git push
   ```

> 提交 PR 前先看对方的 `CONTRIBUTING.md` 和代码风格要求。

---

## 10. 后悔药：撤销修改与提交

按"改到了哪一步"选命令，**先看清再执行**：

| 场景 | 命令 | 影响 |
|---|---|---|
| 文件改乱了，想恢复到上次提交 | `git restore 文件名` | 丢弃工作区改动（不可恢复） |
| `git add` 错了，想撤回暂存 | `git restore --staged 文件名` | 只出暂存区，改动还在 |
| 提交说明写错（**还没推送**） | `git commit --amend -m "新说明"` | 改写最近一次提交 |
| 提交错了（还没推送），想撤销提交但保留改动 | `git reset --soft HEAD~1` | 回到暂存状态 |
| 提交错了（已推送） | `git revert <提交号>` | 生成一条反向提交，安全 |
| 想看看自己刚才干了什么 | `git reflog` | 找回误删的提交 |
| 彻底回到某次提交（**危险**） | `git reset --hard <提交号>` | 丢弃之后所有改动 |

原则：**已经推到 GitHub 的提交，不要用 `reset --hard` 改写历史**，用 `revert` 或新提交修正。

---

## 11. 常见报错速查表

| 报错 | 原因 | 解决 |
|---|---|---|
| `'git' 不是内部或外部命令` | PATH 没生效（本会话环境变量被裁剪） | 新开一个终端；或临时执行 `$env:Path += ';D:\Git\cmd'` |
| `fatal: not a git repository` | 当前目录不是仓库 | `cd` 到仓库目录，或先 `git init` |
| `remote origin already exists` | 已经关联过 origin | `git remote set-url origin <新地址>` |
| `! [rejected] main -> main (non-fast-forward)` | 远程有本地没有的提交（常见于云端初始化了 README） | `git pull --rebase origin main` 后 `git push` |
| `fatal: refusing to merge unrelated histories` | 两段历史互不相干 | `git pull origin main --allow-unrelated-histories` |
| `Authentication failed` / `could not read Username` | HTTPS 用密码了；或凭据失效 | 改用 SSH；或生成 PAT 当密码；`git credential-manager` 里清旧凭据 |
| `Permission denied (publickey)` | 公钥没加到 GitHub / 加错账号 | 重新复制 `.pub` 内容到 <https://github.com/settings/keys>；`ssh -vT git@github.com` 排查 |
| `Connection timed out` / `Connection reset`（github.com:443） | 网络访问受限 | 用本机已配好的 SSH 443 方案；或开代理并配 `http.proxy` |
| `warning: LF will be replaced by CRLF` | Windows 换行符自动转换 | 正常提示，无需处理（本机 `core.autocrlf=true`） |
| 中文文件名显示成 `\344\275\240` | 终端转义 | 已全局设置 `core.quotepath=false`，无需处理 |
| `file is 123.00 MB; this exceeds GitHub's file size limit of 100.00 MB` | 单文件超 100MB | 用 Git LFS，或别提交大文件（本机 `.gitignore` 已挡常见压缩包） |
| `Filename too long` | 路径超长 | 已开 `core.longpaths=true` |
| 推送很慢/卡住 | 网络 | 见第 3 节；SSH 已走 443 |

排查万能三连：

```powershell
git status                      # 当前状态
git remote -v                   # 远程地址对不对
ssh -vT git@github.com          # SSH 细节（-v 看握手过程）
```

---

## 12. 本机现状备忘

| 项目 | 值 |
|---|---|
| Git 版本 | 2.56.0.windows.2 |
| Git 安装目录 | `D:\Git`（`git` 已在系统 PATH 中） |
| 全局用户名 | 宁致远 |
| 全局邮箱 | 13581661445@163.com |
| 默认分支 | `main` |
| 本地仓库 | `D:\宁致远\git-repo` |
| 首个提交 | `4821a6b chore: 初始化专属 Git 仓库` |
| SSH 私钥 | `C:\Users\nssd\.ssh\id_ed25519`（无口令） |
| SSH 配置 | `C:\Users\nssd\.ssh\config`（github.com 走 443） |
| 凭据管理器 | Git Credential Manager（HTTPS 弹窗登录用） |

速查命令：

```powershell
git -C D:\宁致远\git-repo status      # 不切换目录直接看状态
git -C D:\宁致远\git-repo log --oneline -5
```

---

## 13. 接下来 30 分钟清单

- [ ] 解决网络：打开 Watt Toolkit/代理，能正常访问 <https://github.com>
- [ ] 注册账号，验证邮箱，用户名定为：__________
- [ ] 复制公钥，添加到 <https://github.com/settings/keys>
- [ ] 终端执行 `ssh -T git@github.com`，看到 `Hi ... successfully authenticated`
- [ ] 网页新建空仓库 `git-repo`（**不勾** README/.gitignore/license）
- [ ] 本地执行 `git remote add origin git@github.com:你的用户名/git-repo.git`
- [ ] `git push -u origin main`，网页看到 README
- [ ] 本地改一行 README，跑一遍 `status → add → commit → push`
- [ ] 看一眼 `git log --oneline`，确认提交出现在历史里

---

## 14. 进阶方向

| 方向 | 说明 | 入口 |
|---|---|---|
| **Issues** | 任务/Bug 清单，可关联提交 | 仓库页 Issues 标签 |
| **GitHub Actions** | 自动跑测试、自动打包、自动部署 | 仓库内 `.github/workflows/*.yml` |
| **GitHub Pages** | 把仓库变成静态网站（免费域名） | Settings → Pages |
| **Releases** | 给版本打标签、附下载文件 | 仓库页 Releases |
| **Stars / Watch** | 收藏、订阅感兴趣的项目 | 仓库页右上角 |
| **Git 别名** | 少敲几个字 | `git config --global alias.st status` |
| **`.gitattributes`** | 精细控制换行符、二进制文件 | 仓库根目录 |

常用别名一键配置：

```powershell
git config --global alias.st "status -sb"
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --decorate -15"
```
