# Git 协作与提交约定

本模板采用日常直接在 `main` 分支开发、提交和同步的轻量流程。工作分支与 Pull Request 按需使用；具体项目如有分支保护规则，以项目设置为准。

## 首次设置

在项目目录执行一次，使 `git pull` 默认通过 rebase 整合远程更新：

```bash
git config pull.rebase true
```

此设置仅作用于当前本地仓库，每位成员克隆后自行设置。

## 日常流程

开始开发前同步代码：

```bash
git switch main
git pull
```

完成改动和相应检查后，提交并推送：

```bash
git status
git add <本次修改的文件>
git commit -m "doc :补充测试规范"
git pull
# 检查同步后的代码和验证结果
git push
```

推送时如遇远程新提交，重新 `git pull`、检查并推送。拉取前先提交本地改动；临时未提交的工作可用 `git stash push` 保存，拉取后用 `git stash pop` 恢复并检查冲突。

## 提交信息

格式为 `<类型> :<简短清晰的描述>`，类型后一个空格，使用英文半角冒号。每次提交聚焦一件事。

| 类型 | 用途 | 示例 |
| --- | --- | --- |
| `feat` | 新增功能 | `feat :增加设备状态页面` |
| `fix` | 修复问题 | `fix :修复状态未更新的问题` |
| `doc` | 文档变更 | `doc :补充测试规范` |
| `style` | 格式或样式调整 | `style :统一代码缩进` |
| `refactor` | 重构代码 | `refactor :整理数据处理逻辑` |
| `chore` | 配置、依赖或工具调整 | `chore :更新构建配置` |
| `merge` | 手动创建合并提交 | `merge :合并功能分支` |

## 遇到冲突

`git pull` 提示 rebase 冲突时，打开冲突文件并保留需要的内容，删除冲突标记后执行：

```bash
git add <已解决冲突的文件>
git rebase --continue
```

如仍有冲突，重复处理直到完成；验证后再推送。需要取消本次 rebase 时使用 `git rebase --abort`。

不要强制推送覆盖其他人的提交，也不要提交密钥、访问令牌或本地构建产物。
