# Git

## Git分支相关命令、实际引用

### 分支

> 创建分支

```sh
git branch <分支名>

git branch -v 查看分支
```

> 切换分支

```sh
git checkout <分支名>

一步完成: git checkout -b <分支名>
```

> 合并分支

```sh
先切换到主干 git checkout master

git merge <分支名>
```

> 删除分支

```sh
先切换到主干 git checkout master

git branch -D <分支名>
```

## Git 工作流

![](http://120.77.237.175:9080/photos/eight/other/01.png)

