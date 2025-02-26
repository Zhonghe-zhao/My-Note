
五种分支：

Master 主干分支
Develop 开发分支
Feature 特性分支
Release 发布分支
Hotfix 热修复分支

## github工作流

在Fork的仓库进行开发

在github设置仓库的保护分支

	Fast-forward合并方式：
```shell

git chekout -b test

vim readme

git comit -m "test"

git checkout main

git merge test --ff-only
```

	三方合并
git merge test --no-ff