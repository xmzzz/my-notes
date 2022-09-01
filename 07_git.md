git config --global user.name "YourName"
git config --global user.name
git config --global user.email "xxx@xxx.com"
git config --global user.email
git diff > xxx.patch
git config --global core.editor vim
git format-patch -1 2691e0267 -o ~/path/

# github pull request
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


# git patch
git format-patch -1 2691e0267 -o ~/path/

# git log 
git log -p -1 185143bd
git log --oneline

# After PR merged, delete dev branch, syn local repo
git branch -a
git remote show origin
git remote prune origin
git branch -a
git branch -D dev-fix-issue

# push origin 
git push origin A:A
git push origin A  // when the branch names are the same

# log && show diff
git log -p -2

# show one file's change
git log -p filepath

# show changed file list
git log --stat

# show changed file list at one commitId
git log --stat commitId  ||  git show --stat commitId

# show one file's relative commit
git log -- foo.py bar.py

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
