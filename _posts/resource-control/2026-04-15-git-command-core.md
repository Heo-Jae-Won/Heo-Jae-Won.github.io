---
title: "git command"
published: true
categories: [resource-control]
---

## <span style="color:#802548">_clone branch_</span>

- branch clone

```shell
git clone -branch <branchname>  <remote-repo-url>
```

- getting remote info
- updating remote branch info
- move to there after clone

```shell
git fetch origin
git branch -r   
git switch [remote branch]
```

## <span style="color:#802548">_when branch is deleted by remote and sustained in local_</span>

- just fetch origin is not enough. needs prune

```shell
git fetch origin --prune
git branch -d [branch] 
```

## <span style="color:#802548">_when do git pull with my branch remote_</span>

```shell
git pull --rebase --autostash
## if error happens, rather to use merge
git rebase --abort
git pull
```

## <span style="color:#802548">_when do git pull with dev remote_</span>

```shell
git pull origin dev --rebase --autostash
## if error happens, rather to use merge
git rebase --abort
git pull
```

## <span style="color:#802548">_branch manipulation_</span>
- make new branch and move
- delete remote and local branch both
- rename branch 
- see default branch

```shell
git switch -c [branch]

git push origin --delete [branch이름]

git branch -m [current branch] [future branch name]

git remote show origin
```


## <span style="color:#802548">_branch repulishing when divergence wrong_</span>

```shell
git branch -d [branch name]
git push origin --delete [branch이름]

git switch [branch that should be origin]
git switch -c [new featrue branch]
```

## <span style="color:#802548">_remove local change_</span>

```shell
git restore [filename]
```

## <span style="color:#802548">_remove local change untracked_</span>

```shell
git clean -fdn
```

## <span style="color:#802548">_stash untracked and staged_</span>

```shell
git stash push -u -m "message"
```


## <span style="color:#802548">_adding file staging area_</span>

- add file in staging area for only update and delete

```shell
git add -u
```

- add file in staging area for all

```shell
git add .
```

- removing all staged file

```shell
git restore --staged .
```

## <span style="color:#802548">_add file to just before commit_</span>

- no need to increase commit for adding file

```shell
git add [file]
git commit --amend -m "message" # or git commit --amend  
```

- removing just before commit, preseving file on worksapce

```shell
git reset --mixed
```

## <span style="color:#802548">_when u want to do experiment about bugs using completely different branch_</span>

```shell
git stash push -m "my work before fixing a bug"

git switch -c fix/feature

git add .
git commit -m "implement fix"

git fetch origin
git rebase origin/main

git push -u origin fix/feature

# open GitHub PR (fix/feature → main)

git switch main
git branch -d fix/feature

git stash drop stash@{0}   # only if you still have stash left
```

- simplified version that dont use rebase

```shell
git switch main
git pull
git switch -c fix/feature

# work + commit
git add .
git commit -m "fix bug"

git push -u origin fix/feature
# open PR

git switch main
git branch -d fix/feature
```

## <span style="color:#802548">_git command customizing_</span>

```shell
vim ~/.gitconfig:
```
- it looks like below
    - ls is showing commit log pretty
    - rs is checking cleared stash list
    - dl is checking commit log under some condition 
    - fl is showing file list that i develope from certain date

```text
[user]
        email = 222
        name = 333
[alias]
        lo = log --oneline
        ls = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate
        rs = fsck --no-reflog | awk '/dangling commit/ {print $3}' | xargs -L 1 git --no-pager show -s --format="%ci %H" | sort
        dl = git log --name-only --date=local --author=Heo-Jae-Won --since="2024-01-03"
        fl = git log --name-only --author=Heo-Jae-Won --date=local --since="2024-01-03" | egrep -v "^commit|^Date:|^Merge:|^Author:|^\s" | sort | uniq
```

- if we execute this one, we can show this one

```text
git config --list

alias.ls=log --pretty=format:%C(yellow)%h%Cred%d\ %Creset%s%Cblue\ [%cn] --decorate
alias.ll=log --pretty=format:%C(yellow)%h%Cred%d\ %Creset%s%Cblue\ [%cn] --decorate --numstat
.
.

```


## <span style="color:#802548">_git username email config_</span>

```shell
git config user.name [username]
git config user.email [email]
```


## <span style="color:#802548">_seeing previous commit content_</span>

```shell
git restore --source HEAD~0 [파일이름] ---> current file. useless
git restore --source HEAD~1 [파일이름] ---> 1 commit before file.
git restore --source HEAD~2 [파일이름] ---> 2 commit before file.

git restore [파일이름]---> go back to original HEAD 
```

## <span style="color:#802548">_HEAD~ and HEAD^ diff for merge commit_</span>

- HEAD~ is about commit history
    - HEAD~1 = C
    - HEAD → C → B
    - HEAD~2 = B
    - HEAD~3 = A
- HEAD^ is about commit parent
    - HEAD^ = HEAD^1 = C
    - HEAD^2 = F

```text
      E -- F
     /      \
A -- B -- C -- D (HEAD)
```


## <span style="color:#802548">_cherry pick from dev to main_</span>

- let's suppose below commit exists

```shell
cderf21 --> HEAD

rtqwrr51 --> HEAD~2
```

- below is dangerous, cuz cherry-pick picks out just diff. 
- so, if target branch doesnt have the file needed for commit cdef21, conflicts happends. 
- because file is created in commit rtqwrr51. so it's safer to cherry-pick range.

```shell
git cherry-pick cdef21 
```


- i want to get commit from HEAD to HEAD~2, then i must execute it reversely
- from index is not included. so need to be started from one point before

```shell
git cherry-pick rtqwrr51^..cdef21
```

- combining all commit into 1

```shell
git cherry-pick --no-commit rtqwrr51^..cdef21
git commit -m "final combined work"
github PR
```

## <span style="color:#802548">_rebase：replacing cherry pick_</span>

- above one can be replaced by rebase
- but it must be cautious that rebase must be done in local commit, not pushed one

```shell
git rebase -i rtqwrr51^
```

- text would be shown like below

```shell
pick rtqwrr51
pick abc123 --->squash
pick def456 --->squash
pick cdef21 --->squash
git commit --amend -m "Implement feature X (final version)" # (if message editor doesnt open)
git push # (if already published --force-with-lease)
```

- if it's already published in remote, teammates should do following one whether it's used on my own or shared

```shell
git fetch
git rebase origin/mybranch
```

## <span style="color:#802548">_git conflict on branch when using git pull_</span>

```shell
# fixing content
git add [fixed file]
git merge --continue
```

- structure would be like below

```sh
M = merge commit (newly created)
D is NOT rewritten
X → Y is NOT rewritten
Git combines both histories


A --- B --- C --- D              ---> this is merged version
              \       \
               X --- Y --- M
```

## <span style="color:#802548">_rebase：fix local commit message before publishing to remote_</span>

```shell
git rebase -i HEAD~n :
pick -> reword 
change commit message -> :wq
```


## <span style="color:#802548">_rebase：PR_</span>

- in my feature branch is executed
- only for local commit

```shell
git fetch origin
git rebase -i origin/dev #(if target is main, origin/main)
```

- if conflicts happens
- after rebase, already aligned commit. so it becoms fast forward --> clean history

```shell
git add
git rebase --continue
git push origin feature/my-work
open PR → feature → dev 
```

- basic form of rebase is like below
    - it means  Git will rebase branch onto upstream
- but latter one is used often
    - it means "Take the current branch (HEAD) and replay its commits on top of origin/dev."

```shell
git rebase <upstream(origin/dev)> <branch(feature/login)>

git rebase origin/dev
```

- so two combination is usally used for rebasing

```shell
git switch feature/branch
git rebase origin/dev
```

## <span style="color:#802548">_reflog：showing history for git conduct_</span>

- HEAD in reflog doesnt mean branch's head
    - therefore, HEAD@{0} and HEAD@{2} can be actually refer to same thing
- reflog show command can limit target of command like below

```shell
git reflog show HEAD@{2.days.ago}
git reflog show HEAD@{1.month.ago}
git reflog show HEAD@{1.week.ago}
git reflog show master@{0} master@{yesterday}
```



## <span style="color:#802548">_reflog：restore file_</span>
- git reflog show --> finding a commit hash that i need to recover
- after restoring, file is recreated in working directory, not inside Git history
- so need to again add and commit

```shell
git log --all -- '*file*'
    commit 8f3a91c2d7b5a4c1e9a0f2b6c7d8e9f1a2b3c4d5
git ls-tree -r <commit-hash>
    100644 blob aaa111 src/app/main.py
    100644 blob bbb222 src/utils/helper.py
git restore --source <commit> <file-path>
```

## <span style="color:#802548">_temp save_</span>

- we can save our work to temp space
- but untracked file cannot be target of stash

```shell
git stash push -m "my work before fixing a bug"
git stash pop
```

## <span style="color:#802548">_revert：commit delete from remote repo_</span>

- let's suppose below commit flow

```sh
A — B — C — D — E — F   (latest = F, commit123 = C)

* f6f6f6f (HEAD -> main) Commit F
* e5e5e5e Commit E
* d4d4d4d Commit D
* c3c3c3c Commit C   <-- commit123
* b2b2b2b Commit B
* a1a1a1a Commit A
``` 

- then commit from C to F is reverted

```sh
git revert --no-commit C^..F

git push origin master
```



## <span style="color:#802548">_log reading_</span>

- origin/HEAD is not a branch HEAD
    - a pointer to the default branch on the remote
    - usually points to origin/main or origin/master
- origin/main is remote repo recent commit
- main is local repo recent commit
- when origin/main and main is together, it means no divergence from local and remote 
- everything up to date.

```shell
de12b0e (origin/main, origin/HEAD, main) commit
```


- local branch is fully up to date with the remote
- HEAD -> means that i am not detached state, which means on this branch
- if no HEAD ->, it means i am detached state

```sh
4c6cfcc (HEAD -> feature/recommend, origin/feature/recommend) commit2
```


- 785cd77 HEAD -> feature/authority-profile doesnt be written remote origin.
    - it means that remote feature/authority-profile has ahead commit, which means needs git pull
    - actually, other developer can commit on my branch if it's shared branch.
- 68aa468 (master) Merge pull request #122 


```sh
785cd77 (HEAD -> feature/authority-profile) Feat: generate role
```

- (feature/dev_wsg) and (origin/feature/dev_wsg) is diverged
- it means local has more ahead commit 

```sh
82174d3 (feature/dev_wsg) fix: exclude deactivated account#2
8339e87 fix: exclude deactivated account
afdfd6d add: search word option(=address)
35d69ef (origin/feature/dev_wsg) adding profile path
```

- 8957fea origin/hotfix/add-db-tls doesnt be written (origin/hotfix/add-db-tls, hotfix/add-db-tls)
    - in this case, origin/hotfix/add-db-tls is not about ahead or behind when there is no hotfix/add-db-tls in commit log
    - it means that i dont publish hotfix/add-db-tls on my local 

```sh
8957fea (origin/hotfix/add-db-tls) add-db-enable-tls
```



## <span style="color:#802548">_merge and ff_</span>

- let's suppose this kind of commit flow

```sh
A ー Bー Cー D. 
``` 


- i diverge branch from main D commit
- commit on my branch is E and F is executed.

```sh
main: A — B — C — D — G
                 \
feature:          E — F
```

- in this case, when using git status below message is shown
- showing ahead and behind simultaneously means diverging of branch 

```sh
1 commit ahead, 2 commit behind 
```

- in this case, ff is impossible
- history diverged at D:
    - main has G
    - my feature has E, F
- so Git cannot just "move the pointer" 
- both branches have unique commits.
- Therefore, fast-forward is impossible
- merge commit is generated

```sh
[merge]
A — B — C — D — G
                 \ 
                  M
                 /
            E — F
```

- if u want flow like below,

```sh
A — B — C — D — G — E' — F'
```

- rebase is needed

```sh
git switch feature
git rebase main
```

- then when ff is possible?
- when no advanced commit on main exists, ff is possible

```sh
main:    A — B — C
feature:          D — E
```

- if merging operation is finished, git status shows like this message

```sh
up to date
```



