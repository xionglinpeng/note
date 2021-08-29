# Github personal access token

2021年8月13，Github删除了对密码验证的支持。如果使用`git pull`或者`giit push`等命令将报错。要推送及拉取项目，需要使用个人访问令牌。  

报错信息如下：

```
Support for password authentication was removed on August 13, 2021. Please use a personal access token instead。
```

### 创建新的access token

1. 右上角头像→`Settings`→`Developer settings`→`Personal access tokens`→`Generate new token`。
2. 日期选择No expiration（没有过期），授权范围全部选上。

如下图所示：

![1630252906628](images/Generate new token.png)

## 使用Token访问Github API

### 命令配置

首先移除原origin远程地址

```shell
$ git remote remove origin
```

添加基于Token访问的新地址

```shell
$ git remote add origin https://[Note]:[token]@github.com/[xionglinpeng]/[project_name].git
```

### 文件配置

使用文本编辑器打开`.git/config`文件（位于项目目录下），修改`[remote "origin"]`下`url`属性。

```properties
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[credential]
	helper = 
[remote "origin"]
	url = https://[Note]:[token]@github.com/xionglinpeng/note.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```













