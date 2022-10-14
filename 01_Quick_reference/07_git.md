# git config 
```
git config --global user.name "YourName"
git config --global user.name
git config --global user.email "xxx@xxx.com"
git config --global user.email
git config --global core.editor vim
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
git diff > xxx.patch
git format-patch -1 2691e0267 -o ~/path/
git format-patch -1 2691e0267 -o ~/path/
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