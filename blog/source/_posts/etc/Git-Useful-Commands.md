---
title: Git Useful Commands
date: 2021-12-15 19:30:00
author: Aaron
cover: true 
toc: true
summary: Common useful git command
categories: 
  - Git
tags: 
  - Git
---


# Git

Git is a version control system on a machine locally.

## Necessary Git Commands

```bash
git init
git status

git log
git log --oneline

git push
git push -f # force push

git branch "branch_name"
git branch -d "branch_name" # delete
git branch -M "new name" # rename current branch

git switch "branch_name"
git switch - # back to whatever branch I was
git switch -c "branch_name" # create a branch and switch to it
git checkout -b "branch_name" # create a branch and switch to it

git add file1, file2, file3
git add .

git commit
git commit -m "message of this commit"
git commit -am "message of this commit" # add all and commit with msg
git commit --amend # modify the most recent commit

git merge
# - Fast Forward Merge
# - Merge Commit
# - CONNFLICT
#     - (0) git merge --abort
#     - 1. git status to find unmerged files
#     - 2. edit files and remove conflict "markers"
#     - 3. git add <file>
#     - 4. git commit
```

## Useful Git Commands
### Git Diff

> to view changes between commits, branches, files, working directory, ...

```bash
# @@ -1,6 +1,8 @@: Chunk Header
#    1. 舊版本，從第1行開始連續6行
#    2. 新版本，從第1行開始連續8行

# 1. 當工作區有改動，暫存區為空，diff的對比是“工作區與最後一次commit提交的倉庫的「共同文件」”
# 2. 當工作區有改動，暫存區不為空，diff對比的是“工作區與暫存區的共同文件”。
git diff

# 工作區和暫存區與最後一次commit(HEAD)之間的變動
git diff HEAD

# 顯示暫存區(staged)和最後一次commit(HEAD)之間的所有不相同文件的增刪改
git diff --staged
git diff --cached

git diff HEAD [filename]
git diff --staged [filename]

git diff branch1..branch2
git diff commit1..commit2
```

### Git Stash

> Git provides an easy way of stashing these uncommitted changes so that we can return to them later, without having to make unnecessary commits.


```bash
# stash all uncommitted changes
git stash
git stash save

# stashed changes are restored
git stash pop

# apply whatever is stashed away
# It can be useful if you want to apply stashed changes to multiple branches
git stash apply # 在不同branch apply，可能要解conflict
git stash apply stash@{2} # apply particular one

git stash list # show stash list

git stash drop # drop top stashed changes only
git stash drop stash@{3} # drop particular one
git stash clear # clear all stashed changes
```
### Undo changes, Go back in time

#### Detached HEAD
```bash
# checkout 功能太多，才衍生 switch, restore
git checkout <commit-hash> # view a previous commit
# 1. Usually, HEAD points to a specific branch reference rather than a particular commit. When we checkout a particular commit, HEAD points at that commit rather than at the branch pointer (detached head)
# 2. Levae by using git switch <branch-name>
# 3. Create a new branch and switch to it. (new branch is based upon HEAD)

git checkout HEAD~2 # refers to 2 commits before this HEAD
```

#### Discarding Changes
```bash
git checkout HEAD <filename> # to discard any changes in that file, reverting back to the HEAD
git checkout -- <filename>
git restore <filename1, filename2, ...> # to restore the file to the contents in the HEAD
git restore --source HEAD~1 <filename> # to restore the contents to its state from the commit prior to HEAD
git restore --source <commit-hash> <filename>
git restore --staged <filename> # remove file from staging area, but it is still in the working area.

git reset <commit-hash> # will reset the repo back to a specific commit, the commits are gone, but the changes are still in the working area
git reset --hard <commit-hash> # will delete the alst commit as well as associated changes


# 1. reset removes commits entirely and it moves the branch pointer backwards if those commits nevers occurred
# 2. revert creates a new commit which undo changes from a commit, revert may also cause CONFLICT
git revert <commit-hash>
# 使用revert而不是reset的時機：假設有一些bad commit要移除，但是在多人合作模式下，若這些commit正在被其他人使用中，我不能直接透過reset移除這些commit，這樣之後merge會有問題，那為了把過去的錯誤消除，我們能透過revert創立新的commit去移除這些問題。
```

# Github

Github is a hosting platform for git repositories

## Necessary Commands

### Setting

```bash
git clone <URL> # URL並不是綁github的連結
git config user.email "mail"
git config user.password ""

# SSH Keys: need to be authenticated on Github to do certain operations
# 1. check whether ssh key exists: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys
# 2. generate a new ssh key: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
```

### Remote
```bash
git remote -v # show url of fetch, push
git remote # origin

# EXIST LOCAL REPO
# 1. Create a new repo on Github
# 2. Connect your local repo (add a remote)
# 3. Push up your changes to Github
git remote <name> <url> # git remote origin https://github.com/...., connect the local repo to a url
git remote rename <oldname> <newname>
git remote remove <name>

# START FROM SCARTCH
# 1. Create a brand new repo on Github
# 2. Clone it down to your machine
# 3. Do some work locally
# 4. Push up your changes to Github
```

### Push
```bash
git push # 會提交到"預設"的儲存庫
git push <remote> <branch> # `git push origin master`, push up the master branch to our origin remote

git push <remote> <local-branch>:<remote-branch> # push local branch into remote branch

git push origin --delete <remote-branch> # delete remote branch
git push origin -d <remote-branch>

git push -u origin master # -u 表示將origin/master設定為本地master分支的upstream
git push -u origin <local-branch>:<remote-branch>
```

### Pull & Fetch
![Pull vs. Fetch](/images/git/pull_fetch.png)
```bash
#當本地branch跟remote的branch不同時，也能checkout remote branch
git checkout origin/main

# 當我們clone一個repo下來，master之外的其他branches並不會被載下來
git branch # 此時local看不到其他branch
git branch -r # 看的到remote的branch
git switch <remote-branch-name> #就能直接track remote branch到local
git checkout --track origin/<remote-branch-name> #舊版的方式去track remoet branch

# fetching allow us to download all changes from the origin remote repo, BUT those changes will not be auto integrated into working files
# 注意fetch並不會讓local working area的檔案被修改，只是更新存在本地端的<remote>/<branch>資訊
git fetch <remote> # fetch all changes from remote, 包含新建的branch
git fetch
git fetch origin 
git fetch <remote> <remote-branch>


# pulling will go and download data from Github AND immediately update my local repo with those changes.
# git pull = git fetch + git merge
# May cause Merge Conflict
git pull <remote> <branch>
# 預設抓origin中同名字的branch
```


# Collaboration

## READMEs
A README file is used to communicate importanat information about a repository including:
- What the porject does
- How to run the porject
- Why it's noteworthy
- Who maintains the project

Markdown
- [markdown-it demo](https://markdown-it.github.io/)

## centralized workflow problem
- Lots of time spnet resolveing conflict and merge code, especially as team size scales up.
- No one can work on anything without disturbing the main codebase. How do you experiment?
- The only way to collaborate on a feature together with another teammate is to push incomplete code to master. Other teammates now have broken code.

## Feature Branches
Rather than working directly on master, all developer should be done on separate branches
```bash
git fetch
git branch -r
git switch origin branch_name
git pull origin branch_name

在master，如果不透過PR，可以直接切到master並
git merge branch_name
```

## Pull Request
to alert team members to new work that needs to be reviewed.
1. Do some work locally on a feature branch then push up the feature.
2. Open a Pull Request using the feature branch just pushed up to Github
3. Wait for the PR to be approved and merged.