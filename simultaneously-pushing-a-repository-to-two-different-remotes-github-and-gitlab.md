---
description: 将一个仓库同时推送到2个不同的远端，一个GitHub，一个GitLab
---

# Simultaneously Pushing a Repository to Two Different Remotes: GitHub and GitLab

有时候我们需要将一个仓库同时推送到两个不同的远端（GitHub和GitLab），可以按照以下步骤进行操作：

1. 先在 Github 上创建一个空的仓库
2. 本地 clone 这个仓库，然后初始化一些信息
3. git push 到 Github 远端仓库
4. 在 Gitlab 上创建一个同名的空仓库，获取此 Gitlab 仓库的 clone 地址
5. 在刚才的本地仓库中添加

```typescript
git remote add gitlab <GitLab remote Url>
```

6. 这时候再次使用 `git remote -v` 可以查看到当前仓库已经分别关联了 Github 和 Gitlab



当你要推送到 Github 时， 直接使用默认的 `git push` 即可

当你想要推送到 Gitlab 时，使用 `git push gitlab` 推送



另外确保两边创建时都为空仓库，保持 branch 名的统一。

