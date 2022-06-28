# Git 常规操作

## 切换分支
```shell
➜  istio git:(master) ✗ git checkout -b release-1.9 origin/release-1.9
error: Your local changes to the following files would be overwritten by checkout:
	pilot/pkg/xds/testdata/benchmarks/telemetry.extra.yaml
Please, commit your changes or stash them before you can switch branches.
Aborting
➜  istio git:(master) ✗ 
➜  istio git:(master) ✗ 
➜  istio git:(master) ✗ 
➜  istio git:(master) ✗ git stash

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'root@biz-master-48.(none)')
Cannot save the current index state
➜  istio git:(master) ✗ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	typechange: pilot/pkg/xds/testdata/benchmarks/telemetry.extra.yaml
#
no changes added to commit (use "git add" and/or "git commit -a")
➜  istio git:(master) ✗ git config --global user.email "leis17@163.com" 
➜  istio git:(master) ✗ git config --global user.name "mark"          
➜  istio git:(master) ✗ 
➜  istio git:(master) ✗ 
➜  istio git:(master) ✗ 
➜  istio git:(master) ✗ git stash                                      
Saved working directory and index state WIP on master: e795de5 Automator: update proxy@master in istio/istio@master (#39630)
HEAD is now at e795de5 Automator: update proxy@master in istio/istio@master (#39630)
➜  istio git:(master) 
➜  istio git:(master) 
                                                                                                                                        
➜  istio git:(master) 
                                                                                                                                        
➜  istio git:(master) 
➜  istio git:(master) git checkout -b release-1.9 origin/release-1.9 
Branch release-1.9 set up to track remote branch release-1.9 from origin.
Switched to a new branch 'release-1.9'
➜  istio git:(release-1.9)
```

