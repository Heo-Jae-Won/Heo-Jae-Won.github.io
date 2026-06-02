---
title: "git command"
published: true
categories: [resource-control]
---

## <span style="color:#802548">_git rebase miss caution_</span>

- if u want to pull rebase, then dont need to fetch - rebase
- just pull with rebase option is enough

```shell
git pull origin develop --rebase -p --autostash -->O
git pull origin/develop --rebase -p --autostash-->X
```

## <span style="color:#802548">_how can i know where my branch is from_</span>

- if u are not sure which branch is ur original branch, below is recommended
- but the problem is that, it shows commit even if my branch is from develop/AService
    - cuz develop/AService is from develop
    - but dont worry. if ur branch is from develop, then git merge-base --fork-point [develop/AService] doesnt show commit

```shell
git merge-base --fork-point [develop/AService]
git merge-base --fork-point [develop]
```

- below is the process
- if ur branch is from develop/AService, u can see Z commit

```text
develop          o---o---A---B---C (develop keeps moving)
                         \
develop/AService          X---Y---Z (AService split from A)
                                   \
your-branch                         1---2 (You split from Z)
```

- if ur branch is from develop, u can see C commit

```text
develop          o---o---A---B---C (develop keeps moving)
                                   \
your-branch                         1---2 (You split from C)
```

```
git show-branch
```

## <span style="color:#802548">_update remote branch_</span>


- getting remote info
- updating remote branch info
- move to there after clone

```shell
git fetch origin --prune
git switch [remote branch]
```

## <span style="color:#802548">_when branch is deleted by remote and sustained in local_</span>

- just fetch origin is not enough. needs prune

```shell
git fetch origin --prune
git push origin [branch이름] --delete
git branch -D [branch이름] 
```

## <span style="color:#802548">_merge build error fix commit to original service logic_</span>

- suppose that i commited 2 different things
    - build error fix
    - service logic
- order is service -> build error

```
72a1    fix build error
392c    service logic
```

- but i conclude that fix build error commit is not needed
- then do rebase and fixup

```sh
git rebase -i HEAD~2
```

- then below window is displayed

```shell
pick 392c service logic
pick 72a1 fix build error
```

- leave what u want to preserve
- in this case, i decided to preserve 392c

```shell
pick 392c
fixup 72a1
```

- then, build error fix commit's content is merged into 392c, which is service logic commit
- to ensure build error fix commit is merged, git log

```shell
392c service logic
```


## <span style="color:#802548">_when do git force push, rejected_</span>
```
[rejected] branch -> branch(stale info) 
```

- this occurs cuz u pushed another commit from another PC.
- so u need to reset hard and pull ur own bracnh.

## <span style="color:#802548">_remove local change_</span>

- remove local change tracked and untracked both

```shell
git restore .
git clean -fdx
```

- removing all staged file

```shell
git restore --staged .
```

## <span style="color:#802548">_stash untracked and staged_</span>

```shell
git stash push -u -m "message"
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

## <span style="color:#802548">_reflog：showing history for git conduct_</span>

- HEAD in reflog doesnt mean branch's head
    - therefore, HEAD@{0} and HEAD@{2} can be actually refer to same thing

```shell
git reflog
git reset --hard [commit hash that i want to get back]
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



## <span style="color:#802548">_credential manager initialization_</span>

- if u want to initialize credential manager, u need to do it on below path

```text
Control Panel → Credential Manager → Windows Credentials
```

- then look for entries like:

```text
git:https://dev.azure.com/...
MicrosoftAccount
Azure DevOps
```

- if u found it, then remove that credentials and try to git fetch
- then, new login authentication message is displayed.
