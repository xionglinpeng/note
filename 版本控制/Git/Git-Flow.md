# Git Flow





![](https://upload-images.jianshu.io/upload_images/10439291-1a03ba780894a139.png?imageMogr2/auto-orient/strip|imageView2/2/w/614/format/webp)



## 分支定义

Git Flow对于分支的命名有严格的定义：

- master — 只能用来包括产品代码。一般开发人员不能直接工作在这个master分支上，而是在其他指定的、独立的特性分支中。
- develop — 进行任何新的开发的基础分支。当开始一个新的功能分支时，它将是开发的基础。另外，该分支也汇集所有已经完成的功能，并等待整合到master分支中。
- feature — 基础develop分支所检出的用于开发特性功能的分支。
- hotfix — 基于产品代码（一般是指master分支）检出的用于修复产品BUG的分支。
- release — 基于develop分支所检出的用于发布版本的分支。



```shell
$ git flow
usage: git flow <subcommand>

Available subcommands are:
   init      Initialize a new git repo with support for the branching model.
   feature   Manage your feature branches.
   bugfix    Manage your bugfix branches.
   release   Manage your release branches.
   hotfix    Manage your hotfix branches.
   support   Manage your support branches.
   version   Shows version information.
   config    Manage your git-flow configuration.
   log       Show log deviating from base branch.

Try 'git flow <subcommand> help' for details.
```

在Git 2.7.4版本测试是不支持`git flow`命令

## 初始化Git-Flow

刚开始时一个项目默认只有master分支，通过git flow init名称进行flow模式的初始化，这样后续git flow相关命令时才会知道怎样生成相应的分支信息。命令如下：

```shell
$ git flow init

Which branch should be used for bringing forth production releases?
   - master
Branch name for production releases: [master]
Branch name for "next release" development: [develop]

How to name your supporting branch prefixes?
Feature branches? [feature/]
Bugfix branches? [bugfix/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []
Hooks and filters directory? [${PROJECT_HOME}/.git/hooks]
```

首先会询问：使用哪个分支作为生产版本?  （Which branch should be used for bringing forth production releases?）下面会列出当前已有的分支——如上所示：- master。默认为master，因此在这里直接默认回车进行下一步（如果有其他需要可以在Branch name for production releases: [master]之后输入需要作为生成版本的分支名）。

紧接着，要求输入开发分支的名称（Branch name for "next release" development: [develop]），这里直接保持默认名为develop的分支即可。因为这里只有master分支，所以在初始化完成之后会基于master创建develop分支。另外，后续在创建feature分支的时候，会基于该分支创建。

接下来会询问如何命名您的支持分支前缀?（How to name your supporting branch prefixes?），它的作用是设置在使用git flow时生成相应的feature分支，release分支等其分支的前缀。例如我们将Feature branches? [feature/]设置为abc，那么在执行命令git flow feature start demo时，生成的feature分支名为abc/demo 。因此下面的一般所有都保持默认即可。

**初始存在多个分支的情况**

上面的示例只存在一个master分支，如果存在多个分支，情况稍微有点差异。如下所示：

```shell
$ git flow init

Which branch should be used for bringing forth production releases?
   - dev
   - master
Branch name for production releases: [master]

Which branch should be used for integration of the "next release"?
   - dev
Branch name for "next release" development: [] dev

How to name your supporting branch prefixes?
Feature branches? [feature/]
Bugfix branches? [bugfix/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []
Hooks and filters directory? [${PROJECT_HOME}/.git/hooks]
```

差异在于在要求输入开发分支的名称时，没有默认选择提供，因此这里可以输入一个已经存在或者不存在的分支名。

在没有提供默认名称的情况下，必须输入名称，如果不输入，将会报错，如下所示：

```shell
Fatal: Missing branch name
```

当然，无论是存在多个分支，还是只有一个master分支，其本质都是一样的。

### 重新初始化

已经初始化了，但是需要重新初始化，使用命令`git flow init -f`。如果不加`-f`参数进行初始化，将会有如下提示：

```shell
git flow init
Already initialized for gitflow.
To force reinitialization, use: git flow init -f
```

## 新功能开发工作流

1. 开始新的功能

   ```shell
   $ git flow feature start demo
   Switched to a new branch 'feature/demo'
   
   Summary of actions:
   - A new branch 'feature/demo' was created, based on 'develop'
   - You are now on branch 'feature/demo'
   
   Now, start committing on your feature. When done, use:
   
        git flow feature finish demo
   ```

   执行流程：

   1. 基于`develop`分支创建了一个新的分支`feature/demo`。
   2. 切换到`feature/demo`分支。



执行上述命令，git-flow会创建一个名为“feature/user”分支。其中“feature/”前缀代表我们的分支是一个特性功能的开发。

2. 完成一个功能

   经过一段时间的开发和提交，完成了功能的开发，就需要将对应的feature分支合并回develop分支中：

   ```shell
   $ git flow feature finish demo
   Switched to branch 'develop'
   Updating 2c71900..44909d7
   Fast-forward
    README.md | 2 +-
    1 file changed, 1 insertion(+), 1 deletion(-)
   Deleted branch feature/demo (was 44909d7).
   
   Summary of actions:
   - The feature branch 'feature/demo' was merged into 'develop'
   - Feature branch 'feature/demo' has been locally deleted
   - You are now on branch 'develop'
   ```

   执行流程：

   1. 切换到develop分支。
   2. 合并feature/demo分支。
   3. 删除feature/demo分支。





## Bug修复工作流

对于开发工作而言，BUG总是不可避免的。唯一的办法就是尽早的发现BUG，并修复它。

Git-Flow提供了“hotfix”工作流程，用于修复产品BUG。

1. 创建hotfix

   ```shell
   $ git flow hotfix start demo
   Switched to a new branch 'hotfix/demo'
   
   Summary of actions:
   - A new branch 'hotfix/demo' was created, based on 'master'
   - You are now on branch 'hotfix/demo'
   
   Follow-up actions:
   - Start committing your hot fixes
   - Bump the version number now!
   - When done, run:
   
        git flow hotfix finish 'demo'
   ```

   执行流程：

   1. 基于`master`分支创建了新的分支`hotfix/demo`。
   2. 切换到`hotfix/demo`分支。

   这个命令会创建一个名为'hotfix/demo'的分支，其中''hotfix/'代表分支是一个产品BUG的修复，所以hotfix分支基于'master'分支。

2. 完成hotfix

   ```shell
   $ git flow hotfix finish demo
   Switched to branch 'master'
   Your branch is up-to-date with 'origin/master'.
   Merge made by the 'recursive' strategy.
    violet-test/pom.xml | 2 +-
    1 file changed, 1 insertion(+), 1 deletion(-)
   Switched to branch 'develop'
   Merge made by the 'recursive' strategy.
    violet-test/pom.xml | 2 +-
    1 file changed, 1 insertion(+), 1 deletion(-)
   Deleted branch hotfix/demo (was bead84b).
   
   Summary of actions:
   - Hotfix branch 'hotfix/demo' has been merged into 'master'
   - The hotfix was tagged 'demo'
   - Hotfix tag 'demo' has been back-merged into 'develop'
   - Hotfix branch 'hotfix/demo' has been locally deleted
   - You are now on branch 'develop'
   ```

   执行流程：

   1. 切换到master分支
   2. 合并hotfix/demo分支
   3. 切换到develop分支
   4. 合并hotfix/demo分支
   5. 删除hotfix/demo分支



## 版本发布流

当功能开发完成，并且通过了测试，那么此时就具备了版本发布的条件。



1. 创建release

   当“develop”分支中的代码已经是一个成熟的release版本时，这意味着：

   - 它包括所有新的功能和必要的修复；
   - 它已经被彻底地测试过了；

   如果上述两点都满足，则可以生成一个新的release分支了：

   ```shell
   $ git flow release start 1.0.0
   Switched to a new branch 'release/1.0.0'
   
   Summary of actions:
   - A new branch 'release/1.0.0' was created, based on 'develop'
   - You are now on branch 'release/1.0.0'
   
   Follow-up actions:
   - Bump the version number now!
   - Start committing last-minute fixes in preparing your release
   - When done, run:
   
        git flow release finish '1.0.0'
   ```

   执行流程：

   1. 首先基于`develop`创建了新的分支`release/1.0.0`。
   2. 然后切换到'release/1.0.0'分支。

   注意：release分支是使用版本号命名的。这是一个明智的选择，这个命名方案还有一个很好的附带功能，那就是当我们完成release后，Git-Flow会适当地指定去标记哪些release提交。

2. 完成release

   ```shell
   $ git flow release finish '1.0.0'
   D:/developer kits/Git/usr/bin/gitflow-common: line 79: [: -master: integer expression expected
   Branches 'master' and 'origin/master' have diverged.
   And local branch 'master' is ahead of 'origin/master'.
   Switched to branch 'master'
   Your branch is ahead of 'origin/master' by 2 commits.
     (use "git push" to publish your local commits)
   Merge made by the 'recursive' strategy.
    README.md | 2 +-
    1 file changed, 1 insertion(+), 1 deletion(-)
   Already on 'master'
   Your branch is ahead of 'origin/master' by 5 commits.
     (use "git push" to publish your local commits)
   Switched to branch 'develop'
   Merge made by the 'recursive' strategy.
    README.md | 2 +-
    1 file changed, 1 insertion(+), 1 deletion(-)
   Deleted branch release/1.0.0 (was 4cfece3).
   
   Summary of actions:
   - Release branch 'release/1.0.0' has been merged into 'master'
   - The release was tagged '1.0.0'
   - Release tag '1.0.0' has been back-merged into 'develop'
   - Release branch 'release/1.0.0' has been locally deleted
   - You are now on branch 'develop'
   ```

   执行流程：

   1. 切换到master分支
   2. 合并release分支
   3. 切换develop分支
   4. 合并release分支
   5. 删除`release/1.0.0`分支





## git flow help

```shell
git flow
usage: git flow <subcommand>

Available subcommands are:
   init      Initialize a new git repo with support for the branching model.
   feature   Manage your feature branches.
   bugfix    Manage your bugfix branches.
   release   Manage your release branches.
   hotfix    Manage your hotfix branches.
   support   Manage your support branches.
   version   Shows version information.
   config    Manage your git-flow configuration.
   log       Show log deviating from base branch.

Try 'git flow <subcommand> help' for details.
```





