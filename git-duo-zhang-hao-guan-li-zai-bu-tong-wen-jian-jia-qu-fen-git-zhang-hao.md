# GitHub 多账户管理：在不同文件夹下使用不同账户

处理在同一台电脑下，对不同的仓库使用不同的github账号或者gitlab账号

打开gitconfig文件

```bash
vi ~/.gitconfig
```



添加如下内容：

```bash
[includeIf "gitdir:**/github/**"]
path = ~/.githubconfig
```



新建一个 githubconfig文件

```bash
vi ~/.githubconfig
```



githubconfig内容添加：

```bash
[user]
	name = another_name
	email = another_name@github.com
```



这样以后只要是 github 目录下的仓库，都会走 githubconfig下的账号，可以区分公司账号以及个人账号。

