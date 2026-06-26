# AutoDL 服务器代码推送到 GitHub 全流程：只上传可运行代码，避免大文件

本文记录一次将 AutoDL / Linux 服务器中的深度学习项目代码推送到 GitHub 的完整流程。目标是只上传可以复现实验的核心代码、配置文件和说明文档，不上传数据集、模型权重、训练日志、缓存文件等大文件。

本文适合以下场景：

- 项目代码在云服务器或 AutoDL 服务器上；
- GitHub 仓库已经创建好；
- 需要把服务器中的代码同步到 GitHub；
- 不希望上传数据集、`.pth` 权重、`work_dirs`、训练日志等大文件；
- 使用 HTTPS + GitHub Personal Access Token 进行认证。

## 1. 项目场景

本次服务器项目目录示例为：

```bash
/root/autodl-tmp/HVAC-MFR-github
```

GitHub 仓库地址示例为：

```text
https://github.com/sunyue893567/HVAC-MFR
```

最终目标是将服务器中的轻量化项目代码推送到 GitHub：

```text
https://github.com/sunyue893567/HVAC-MFR
```

注意：仓库中只保留可以运行和复现实验的代码、配置和文档，不上传大体积文件。

## 2. 建议上传的内容

深度学习项目建议上传以下内容：

```text
README.md
LICENSE
.gitignore
configs/
mmseg/
tools/
docs/
requirements.txt
```

如果是基于 MMSegmentation 的自定义模型，通常至少需要上传：

```text
configs/hvac_mfr/
mmseg/models/backbones/
mmseg/models/decode_heads/
README.md
```

这些内容可以让别人根据 README 重新配置环境、准备数据集并运行训练、验证和推理。

## 3. 不建议上传的内容

以下内容不建议上传到 GitHub：

```text
data/
datasets/
work_dirs/
outputs/
runs/
pretrain/
checkpoints/
*.pth
*.pt
*.ckpt
*.onnx
*.log
*.pid
*.zip
*.tar
*.tar.gz
*.rar
__pycache__/
```

原因：

- 数据集通常体积很大，并且可能涉及授权；
- 训练权重和 checkpoint 文件体积很大；
- 训练日志和临时输出不属于核心代码；
- 缓存文件会影响仓库整洁性。

## 4. 登录服务器

如果代码在远程服务器，需要先从本地终端登录服务器：

```powershell
ssh -p 26183 root@connect.westc.seetacloud.com
```

登录成功后，进入项目目录：

```bash
cd /root/autodl-tmp/HVAC-MFR-github
```

确认当前目录是否正确：

```bash
pwd
ls
```

## 5. 配置 `.gitignore`

进入项目根目录后，检查是否存在 `.gitignore`：

```bash
ls -a
```

如果没有 `.gitignore`，可以新建一个。推荐内容如下：

```gitignore
# Python
__pycache__/
*.py[cod]
*.so
*.egg-info/
.pytest_cache/

# IDE and system files
.DS_Store
.vscode/
.idea/

# Datasets and outputs
data/
datasets/
work_dirs/
outputs/
runs/
results/

# Pretrained weights and checkpoints
pretrain/
checkpoints/
*.pth
*.pt
*.ckpt
*.onnx

# Logs and temporary files
*.log
*.pid
*.tmp
*.bak

# Compressed files
*.zip
*.tar
*.tar.gz
*.rar
```

配置完成后，Git 会自动忽略这些大文件和临时文件。

## 6. 检查是否有大文件被误加入

提交之前，建议检查项目中是否存在较大的文件。例如检查大于 1MB 的文件：

```bash
find . -path './.git' -prune -o -type f -size +1M -printf '%p %s bytes\n'
```

如果输出中出现 `.pth`、`.pt`、数据集图片、压缩包、日志文件等，需要确认是否应该排除。

如果文件已经被 Git 跟踪，即使后来写入 `.gitignore`，也需要从 Git 暂存中移除：

```bash
git rm --cached 文件路径
```

例如：

```bash
git rm --cached work_dirs/train.log
git rm --cached pretrain/model.pth
```

这里只是从 Git 跟踪中移除，不会删除服务器上的真实文件。

## 7. 检查 Git 仓库状态

进入项目根目录后，执行：

```bash
git status --short --branch
```

如果当前目录是 Git 仓库，会看到类似：

```text
## main...origin/main
 M README.md
?? configs/hvac_mfr/
```

如果出现：

```text
fatal: not a git repository (or any of the parent directories): .git
```

说明当前目录不是 Git 仓库，需要进入正确目录，或者初始化 Git 仓库。

## 8. 初始化 Git 仓库并绑定 GitHub

如果项目目录还不是 Git 仓库，可以执行：

```bash
git init
git branch -M main
```

然后绑定远程 GitHub 仓库：

```bash
git remote add origin https://github.com/sunyue893567/HVAC-MFR.git
```

检查远程仓库地址：

```bash
git remote -v
```

正常情况下会看到：

```text
origin  https://github.com/sunyue893567/HVAC-MFR.git (fetch)
origin  https://github.com/sunyue893567/HVAC-MFR.git (push)
```

如果远程地址已经存在但写错了，可以修改：

```bash
git remote set-url origin https://github.com/sunyue893567/HVAC-MFR.git
```

## 9. 添加需要上传的文件

建议不要直接 `git add .`，尤其是深度学习项目，容易把数据集、权重、日志文件一起加入。

可以按目录添加：

```bash
git add README.md
git add .gitignore
git add LICENSE
git add configs
git add mmseg
git add tools
git add docs
```

如果项目只需要上传部分自定义代码，也可以更精确地添加：

```bash
git add configs/hvac_mfr
git add mmseg/models/backbones/hvac_mfr.py
git add mmseg/models/decode_heads/hvac_mfr_head.py
git add README.md
git add .gitignore
```

添加后再次检查：

```bash
git status --short
```

确认没有 `.pth`、数据集、日志等大文件被加入。

## 10. 提交代码

确认文件无误后执行提交：

```bash
git commit -m "Add HVAC-MFR implementation and configs"
```

如果只是更新 README，可以使用：

```bash
git commit -m "Update README with environment and usage instructions"
```

如果提示没有可提交内容：

```text
nothing to commit, working tree clean
```

说明当前代码已经提交过，或者还没有添加新的修改。

## 11. 创建 GitHub Personal Access Token

GitHub 现在不支持使用账号密码进行 `git push`，需要使用 Personal Access Token。

进入 GitHub 页面：

```text
Settings -> Developer settings -> Personal access tokens -> Fine-grained tokens
```

创建 token 时推荐填写：

```text
Token name: HVAC-MFR
Resource owner: 自己的 GitHub 账号
Expiration: 可以选择 30 days 或自定义时间
Repository access: Only select repositories
Selected repository: HVAC-MFR
```

权限选择：

```text
Repository permissions:
  Contents: Read and write
  Metadata: Read-only
```

然后点击：

```text
Generate token
```

注意：token 只会显示一次，生成后需要立即复制保存。不要把 token 写进代码、README、CSDN 文章或公开仓库。

## 12. 推送到 GitHub

进入项目目录：

```bash
cd /root/autodl-tmp/HVAC-MFR-github
```

执行：

```bash
git push -u origin main
```

如果已经设置过 upstream，后续可以直接：

```bash
git push
```

终端会提示：

```text
Username for 'https://github.com':
```

这里输入 GitHub 用户名，例如：

```text
sunyue893567
```

然后终端会提示：

```text
Password for 'https://sunyue893567@github.com':
```

这里不要输入 GitHub 登录密码，而是粘贴刚才生成的 GitHub Personal Access Token。

注意：粘贴 token 时终端通常不会显示任何字符，这是正常现象。粘贴后直接回车即可。

推送成功后，会看到类似输出：

```text
Enumerating objects: 21, done.
Counting objects: 100% (21/21), done.
Delta compression using up to 128 threads
Compressing objects: 100% (13/13), done.
Writing objects: 100% (18/18), done.
Total 18 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), done.
To https://github.com/sunyue893567/HVAC-MFR.git
   e739d09..7b7032f  main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

这表示代码已经成功推送到 GitHub。

## 13. 检查是否推送成功

推送后检查本地和远程是否同步：

```bash
git status --short --branch
```

如果看到：

```text
## main...origin/main
```

并且没有 `ahead`，说明本地和 GitHub 已经同步。

也可以查看最近提交：

```bash
git log --oneline -5
```

最后打开 GitHub 仓库页面刷新：

```text
https://github.com/sunyue893567/HVAC-MFR
```

确认 README、配置文件、模型代码是否已经更新。

## 14. 常见问题

### 14.1 Password authentication is not supported

如果报错：

```text
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed
```

原因是 GitHub 不支持使用账号密码进行 Git 操作。

解决方法：

- Username 输入 GitHub 用户名；
- Password 输入 GitHub Personal Access Token；
- 确认 token 有目标仓库的 `Contents: Read and write` 权限。

### 14.2 git push 很久没有反应

如果执行：

```bash
git push
```

后长时间没有输出，可能是网络或认证卡住了。

可以按：

```text
Ctrl + C
```

取消后重新执行：

```bash
git push
```

如果仍然卡住，可以检查网络连接或重新登录服务器。

### 14.3 PowerShell 中 `&&` 报错

如果在 Windows PowerShell 里执行包含 `&&` 的 Linux 命令，可能会报错：

```text
标记“&&”不是此版本中的有效语句分隔符
```

更稳妥的做法是先登录服务器：

```powershell
ssh -p 26183 root@connect.westc.seetacloud.com
```

进入服务器后再执行 Linux 命令：

```bash
cd /root/autodl-tmp/HVAC-MFR-github
git status
git push
```

### 14.4 fatal: not a git repository

如果报错：

```text
fatal: not a git repository (or any of the parent directories): .git
```

说明当前目录不是 Git 仓库。

解决方法：

```bash
cd /root/autodl-tmp/HVAC-MFR-github
git status
```

确认进入正确项目目录后再执行 Git 命令。

### 14.5 本地显示 ahead 1

如果执行：

```bash
git status --short --branch
```

看到：

```text
## main...origin/main [ahead 1]
```

说明本地有 1 个提交还没有推送到 GitHub。

执行：

```bash
git push
```

即可同步。

### 14.6 文件太大无法 push

如果 GitHub 提示文件过大，需要先找出大文件：

```bash
find . -path './.git' -prune -o -type f -size +50M -printf '%p %s bytes\n'
```

如果大文件已经被加入 Git 跟踪：

```bash
git rm --cached 大文件路径
```

然后确认 `.gitignore` 已经排除该类文件，再重新提交：

```bash
git add .gitignore
git commit -m "Remove large files from tracking"
git push
```

## 15. 一套完整命令示例

下面是一套常用流程：

```bash
cd /root/autodl-tmp/HVAC-MFR-github

git status --short --branch

find . -path './.git' -prune -o -type f -size +1M -printf '%p %s bytes\n'

git add README.md
git add .gitignore
git add LICENSE
git add configs
git add mmseg
git add tools
git add docs

git status --short

git commit -m "Update HVAC-MFR code and documentation"

git push
```

如果是第一次推送到远程 main 分支：

```bash
git push -u origin main
```

## 16. 总结

服务器代码推送到 GitHub 的核心流程可以概括为：

```text
进入项目目录
-> 配置 .gitignore
-> 检查大文件
-> git add 需要上传的代码
-> git commit
-> 使用 GitHub token 认证
-> git push
-> 刷新 GitHub 仓库检查结果
```

对于深度学习项目，最重要的是不要把数据集、训练权重、checkpoint、日志和临时输出上传到 GitHub。仓库中保留代码、配置文件和清晰的 README，就可以让其他读者根据说明重新配置环境并运行实验。
