# git config settings

```
$ git config --list
$ git config --get user.email
$ git config --list --show-origin
``` 

# linux kernel | send patch to the mailing list

- 前期配置

```
$ git config --global --edit
[sendemail]
	smtpserver = mail.example.org
	smtpuser = you@example.org
	smtpencryption = ssl
	smtpserverport = 465

$ git config --global sendemail.annotate yes
$ git config format.signOff yes
$ git config --global sendemail.smtpPass 'your password'
$ git send-email --to="~mail/mail@.org" HEAD^
$ git send-email --annotate -v2 HEAD^
$ # add timely commentary below ---
$
$
$ git send-email HEAD~3
$ git send-email 209210d
$ git send-email -1 HEAD^^
```

```
Subject: [PATCH v2] Demonstrate that I can use git send-email

---
This fixes the issues raised from the first patch.

your-name | 1 +
1 file changed, 1 insertion(+)
```


- 缓存密码1小时: `git config --global credential.helper 'cache --timeout 3600'`

> http://help.cstnet.cn/changjianwenti/youjianshoufa/xitongcanshu.html

- commit message 每行不超过75?字

```
#vim
:set textwidth=75
```

- 引用其他提交

```
$ git log -1 --abbrev=12 --pretty='format:%h ("%s")' <commitID>
```

- 如果有先前讨论，需要使用标签 `Link: <address>` 指明。lkml要使用lore永久链接：

```
Link: https://lore.kernel.org/r/<Message-Id>/
```

- 修复某commit 引入的问题时，引用不受行长限制：

```
Fixes: <需如上格式化的commit引用>
```

- 补丁和某人有关，添加 `Cc: ` 标签

- `./scripts/checkpatch.pl -g <commit or revision range> 检查代码格式

- `./scripts/get_maintainer.pl -f <file>` 获取可能的 reviewer 和邮件列表

- 所有patch补丁应抄送 linux-kernel@vger.kernel.org ，注意一次不能超过15个补丁

- [git send-email](https://git-send-email.io/)

- 版本变更需要在 `---` 下添加 

# 将一个commit转移到另一个分支上

```
$ git cherry-pick <commit id>
```

# github pubkey

```
https://github.com/<username>.keys
$ curl -O https://github.com/<username>.keys
```
> https://changelog.com/posts/github-exposes-public-ssh-keys-for-its-users

# 签名

```
Signed-off-by: Mingzheng Xing <xingmingzheng@iscas.ac.cn>
```

# 重写历史 | 删除历史

- 当前修改可先暂存

```
$ git stash
```

- 选择要修改的提交

```
$ git rebase -i HEAD~~~
$ # 也可以用 commit ID，注意需要定位需要修改的上一个commitID
$ git rebase -i dfg2jdaf
```

- `HEAD~~~`代表最后三个提交，波浪线符号个数代表提交个数

- 会在编辑器中出现如下信息，将需要修改的commit的pick改为edit。删除的话修改为drop即可。

```
edit d6ccd23 update a, add a new line. 2nd rebase.
pick 45cc625 update a, add 3,4 line. 2nd rebased
pick b2ca1a6 update a, add 5,6 line.

# Rebase 2eaed73..b2ca1a6 onto 2eaed73 (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor

```

- 保存退出后可以修改该提交和修改提交信息

```
$ git add .
$ git commit --amend
```

- 若有冲突需要解决冲突，再执行 `git add .` 和 `git rebase --continue`，【不】需要再`git commit`。

- 中途可用 `git rebase --abort` 取消rebase

- `git reset --hard ORIG_HEAD` 可以恢复到rebase以前的状态

- 最后可恢复第一步暂存的工作 `git stash pop`

参考链接：

> https://backlog.com/git-tutorial/tw/stepup/stepup7_6.html
> https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E5%86%99%E5%8E%86%E5%8F%B2


# 移除子模块的版本控制

- 将独立的驱动git项目移到主内核git项目里之后，遇到了需要移除子模块的版本控制问题

- 首先手动删除了驱动代码里的 `.git\*` 文件，该步骤不确定是否有必要

- 之后执行

```
$ git rm --cached /path/to/files
```

- 此时驱动代码已不再作为子模块进行单独的版本控制，而是作为新增文件加入到主项目里

- 参考链接：https://www.cnblogs.com/Akkuman/p/10911779.html

# clone 并重命名

```
$ git clone https://foo.git newname
```

# 分支相关

- set-url

```
$ git remote set-url origin XXXX.git
```

- 新建远程已有分支

```
$ git checkout -b new_branch origin/xxx # 新建分支并同步远程分支
```

- 和远程仓库建立连接

```
$ git remote add origin https://github.com/xxx/notes.git
```

- 获取远程仓库某分支

```
$ git fetch origin branch_name
```

- 拉取远程分支内容到本地

```
$ git pull origin branch_name
```

- 查看远程所有分支

```
$ git branch -r
```

- 删除本地分支和远程分支

```
$ git branch -d branchName
$ git push origin :branchName
```

- 重命名分支

```
$ git branch -m oldName newName
```

# CAfile fatal

- 如果出现类似下面的 CAfile 相关错误

```
fatal: unable to access 'https://xxx.git/': server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none
```

- 可以尝试下面两个方法之一

```
$ sudo update-ca-certificates
$ export GIT_SSL_NO_VERIFY=1
```

# 删除未跟踪文件

```
$ git clean -d -n	# 空运行
$ git clean -d -f	# 强制删除
$ git clean -d -i	# 交互方式删除
$ git clean -d -n src	# 指定目录
```

# 版本回退和前进

```
$ git reflog
$ git reset --hard commitId
$ git reset --hard HEAD^	# 回退到上一版本
```

# 查看历史改动

```
$ git blame filename    # 每行代码左侧会显示 commit-id
$ git blame -L 60,150 filename    # 可指定行数范围
$ git show commit-id    # 查看详细的 commit 记录
$ git log -S 'deleted_code' filename   # 查看某行代码的删除记录
$ git log --full-history [-3] -- [file path]   # 查看历史 commit 记录，可指定文件或显示条目数量
```

# 带有 submodule 的工程

```
$ git submodule update --init --recursive
```

相当于分别执行

```
$ git submodule init
$ git submodule update
```

# git config 
```
$ git config --global user.name "YourName"
$ git config --global user.name
$ git config --global user.email "xxx@xxx.com"
$ git config --global user.email
$ git config --global core.editor vim
$ git config --list
$ git config --global --unset user.name
```
# pull request
```
git remote add upstream https://github.com/iovisor/bcc.git
git remote -v
	git remote remove upstream
	git remote set-url origin xxx.git
git fetch upstream
git checkout master
git rebase upstream/master
git log
git checkout riscv
git rebase master
git log
git commit --amend
git log
```
# git patch
```
$ git diff > xxx.patch
$ git format-patch -1 2691e0267 -o ~/path/	# 打包指定的 commit
$ git format-patch -3 2691e0267		# 分别打包 commitID 开始的 3 个 commit
```
# git log
```
git log -p -1 185143bd
git log --oneline
git show --stat f91840a32deef5cb1bf73338bc5010f843b01426
git show  f91840a32deef5cb1bf73338bc5010f843b01426
```
# After PR merged, delete dev branch, syn local repo
```
git branch -a
git remote show origin
git remote prune origin
git branch -a
git branch -D dev-fix-issue
```
# push origin 
```
git push origin A:A
git push origin A  // when the branch names are the same
```
# log && show diff
```
git log -p -2
```
# show one file's change
```
git log -p filepath
```
# show changed file list
```
git log --stat
```
# show changed file list at one commitId
```
git log --stat commitId  ||  git show --stat commitId
```
# show one file's relative commit
```
git log -- foo.py bar.py
```
# downloads new data from a remote repository
What's the difference between git fetch and git pull?

```
git fetch origin
```
git fetch really only downloads new data from a remote repository, but it doesn't integrate any of this new data into your working files.

Fetch is great for getting a fresh view on all the things that happened in a remote repository.

Due to it's "harmless" nature, you can rest assured: fetch will never manipulate, destroy, or screw up anything. This means you can never fetch often enough.

```
git pull origin master
```
git pull, in contrast, is used with a different goal in mind: to update your current HEAD branch with the latest changes from the remote server. This means that pull not only downloads new data; it also directly integrates it into your current working copy files. This has a couple of consequences:

Since "git pull" tries to merge remote changes with your local ones, a so-called "merge conflict" can occur. Check out our in-depth tutorial on How to deal with merge conflicts for more information.

Like for many other actions, it's highly recommended to start a "git pull" only with a clean working copy. This means that you should not have any uncommitted local changes before you pull. Use Git's Stash feature to save your local changes temporarily.

	-- from: https://www.git-tower.com/learn/git/faq/difference-between-git-fetch-git-pull

# patch
```
vim ~/.gitconfig [core] editor = vim

git diff [A.cc | commitID] > test.patch

git format-patch -M master  # 当前分支所有超前master的提交

git format-patch [commitID] # 某次提交以后的所有patch

git format-patch --root 4e162b  # 从根到指定提交的所有patch

git format-patch [commitID] .. [commitID]  # 两次提交之间的所有patch

git format-patch -n [commitID]  [-o ~/path/]  # 某次提交及前n次提交

git apply --stat xxx.patch  # 先检查patch文件

git apply --check xxx.patch  # 检查能否应用成功

git apply xxx.patch  # 打补丁

git apply --reject xxx.patch  #自动合入不冲突的代码，同时保留冲突的部分
```
