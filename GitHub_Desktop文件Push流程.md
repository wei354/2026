# 使用 GitHub Desktop 将本地文件 Push 到 GitHub

本文记录如何使用 GitHub Desktop，将本地新增或修改的文件提交并推送到 GitHub 远程仓库。

---

## 1. 打开正确的本地仓库

启动 GitHub Desktop，在左上角确认当前仓库名称。

可以通过以下方式查看当前仓库对应的本地目录：

```text
Repository → Show in Explorer
```

GitHub Desktop 只能检测当前仓库目录中的文件。

如果需要上传的文件不在该目录中，请先将文件复制或移动到仓库目录。

---

## 2. 检查文件是否被识别

文件放入仓库目录后，返回 GitHub Desktop。

正常情况下，左侧 `Changes` 页面会显示：

```text
1 changed file
```

并列出新增或修改的文件。

例如：

```text
CSDN_GitHub代码推送流程.md
```

如果仍显示：

```text
No local changes
```

可以确认：

1. 文件是否确实位于当前仓库目录；
2. 文件是否被 `.gitignore` 忽略；
3. GitHub Desktop 当前选择的仓库是否正确。

也可以在仓库目录中打开终端，执行：

```bash
git status --short --untracked-files=all
```

如果怀疑文件被忽略，可以执行：

```bash
git check-ignore -v "文件名.md"
```

---

## 3. 关于 LF 和 CRLF 提示

GitHub Desktop 可能显示：

```text
This diff contains a change in line endings from 'LF' to 'CRLF'.
```

含义如下：

- `LF`：Linux 常用换行格式；
- `CRLF`：Windows 常用换行格式。

这通常只是换行格式发生变化，不影响 Markdown 文件内容，也不影响正常上传。

若没有特殊的跨平台协作要求，可以直接忽略该提示。

---

## 4. 提交文件到本地 Git 仓库

确认左侧已经勾选需要提交的文件。

在左下角 `Summary` 中填写提交说明，例如：

```text
添加 GitHub 代码推送流程文档
```

然后点击：

```text
Commit 1 file to main
```

其中：

- `Commit` 表示将改动提交到本地 Git 仓库；
- 此时文件还没有上传到 GitHub 网站。

提交完成后，页面通常会显示：

```text
No local changes
```

这表示本地提交已经成功。

---

## 5. 将本地提交上传到 GitHub

### 情况一：远程分支已经存在

如果顶部显示：

```text
Push origin
```

直接点击 `Push origin`。

等待上传完成后，顶部通常会恢复为：

```text
Fetch origin
```

这表示本地提交已经成功同步到 GitHub。

### 情况二：当前分支尚未发布

如果顶部显示：

```text
Publish branch
```

说明当前 `main` 分支还没有发布到远程仓库。

直接点击：

```text
Publish branch
```

GitHub Desktop 会将本地分支和其中的提交上传到 GitHub。

如果弹出登录或仓库设置窗口，需要确认：

- GitHub 登录账号；
- 仓库名称；
- 仓库所属账号；
- 仓库是公开还是私有。

确认后继续发布即可。

---

## 6. 检查是否上传成功

上传完成后，可以点击：

```text
Repository → View on GitHub
```

或者点击页面中的：

```text
View on GitHub
```

浏览器会打开对应的 GitHub 仓库。

刷新页面后，检查目标文件是否已经出现在仓库中。

---

## 7. Commit、Push 和 Publish branch 的区别

### Commit

```text
Commit to main
```

作用：把文件改动保存到本地 Git 仓库。

只执行 Commit，GitHub 网站上还看不到最新文件。

### Push

```text
Push origin
```

作用：把本地已经提交的内容上传到远程 GitHub 仓库。

### Publish branch

```text
Publish branch
```

作用：第一次将本地分支发布到 GitHub 远程仓库。

通常只在分支尚未绑定远程分支时出现。

---

## 8. 后续更新文件的流程

以后修改文件后，只需重复以下步骤：

1. 将文件保存在当前 Git 仓库目录中；
2. 在 GitHub Desktop 的 `Changes` 中检查改动；
3. 填写 `Summary`；
4. 点击 `Commit to main`；
5. 点击 `Push origin`；
6. 打开 GitHub 网页检查结果。

简化流程：

```text
修改文件
  ↓
GitHub Desktop 检测变化
  ↓
填写提交说明
  ↓
Commit to main
  ↓
Push origin
  ↓
GitHub 网页检查
```

---

## 9. 命令行备用方式

如果 GitHub Desktop 无法正常识别或提交文件，也可以在仓库目录中使用命令行：

```bash
git status
git add "文件名.md"
git commit -m "添加文档"
git push origin main
```

如果文件被 `.gitignore` 忽略，但确认仍需上传，可以强制添加：

```bash
git add -f "文件名.md"
git commit -m "添加文档"
git push origin main
```

使用强制添加前，应先确认该文件不包含密码、Token、密钥、数据集、模型权重或其他敏感内容。

---

## 10. 注意事项

上传前建议检查文件中是否包含：

- GitHub Personal Access Token；
- API Key；
- 服务器密码；
- SSH 私钥；
- 数据库账号；
- 个人隐私信息；
- 大型数据集；
- 模型权重；
- 训练日志和缓存文件。

这些内容不应直接上传到公开仓库。

常见需要加入 `.gitignore` 的目录或文件包括：

```gitignore
__pycache__/
*.pyc
.env
*.pth
*.pt
*.ckpt
work_dirs/
logs/
data/
datasets/
```

---

## 11. 本次实际操作结果

本次文件上传过程如下：

1. 将 Markdown 文件放入 GitHub Desktop 当前仓库目录；
2. GitHub Desktop 成功识别为 `1 changed file`；
3. 填写提交说明；
4. 点击 `Commit 1 file to main`；
5. 本地提交完成后显示 `No local changes`；
6. 由于当前分支尚未发布，点击 `Publish branch`；
7. 分支和文件成功上传到 GitHub。
