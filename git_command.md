# git command



## 常用

```shell
git init
git log
git status
git add .
git commit -m "xxxx"
```



```shell
//丢弃工作区的修改，并用最近一次的内容还原到当前工作区（还原对文件中内容的操作，无法对添加文件、删除文件起作用）
git checkout <file_name>

//清空暂存区，将已提交的内容的版本恢复到本地，本地的文件也将被恢复的版本替换（恢复到上一次 commit 后的状态，上一次 commit 后的修改也丢弃）
git reset --hard "commit_ID"

git reset --hard HEAD 		//表示回退到当前版本，HEAD指向当前版本

git remote show origin		//查看远程分支的更新状态

git restore .				//恢复modified的内容

git clean -xdf				//清除untracked files

git checkout -b "123"

git push --set-upstream origin "123"
```





## branch

```sh
查看分支
git branch 							//查看本地分支
git branch -r 						//查看远程分支
git branch -a 						//查看所有分支，包括本地和远程的分支
git branch -d						//删除分子

2）切换分支
git checkout dev 					//切换到dev分支上

如果要切换到远程 remotes/origin/dev 分支
git checkout remotes/origin/dev		//远程分支路径得写全，不可直接 git checkout origin/dev	



3）创建分支
git checkout dev					//在当前分支上创建dev分支
git checkout -b dev					//在当前的分支上新创建的dev分支并切换到新的创建的dev分支上
git checkout -b dev origin/version		//在origin/version上新建dev分支，并切换到dev分支上
git push origin dev					//创建远程dev分支，本地dev分支必须存在


4）查看分支是从哪个分支上创建的
git reflog --date=local --all | grep dev	//查看在dev分支上的操作
git reflog show --date=iso dev

5）删除分支
git branch -d dev					//删除本地dev分支
git push origin --delete dev		//删除远程dev分支

6）分支的合并merge
git merge dev						//将dev分支合并到当前分支（一般是master主分支）or	
git push							//将当前分支代码push到远程分支上

7)查看commit记录
git log --graph						//查看历史提交记录
```



## git push

```shell
git push											//默认推送到远程master分支上
git push origin <branch-name>:<remote-branch-name>	//推送本地指定分支到远程指定分支上
git push --set-upstream origin <remote-branch-name>	//推送并绑定，以后就可以直接git push
```



## git fetch	git pull

```shell
//git fetch

//git fetch只会更新本地远程分支，不会更新本地分支
git fetch (origin)							//远程主机的更新全部拉取到本地
git fetch origin <remote-branch-name>		//拉取远程特定分支
//git log里没有origin/master，origin/HEAD，是因为origin/master分支有更新，fetch后本地远程分支状态超前于本地分支，所以无法显示
git merge origin/master		//fetch的远程分支和本地合并
```



```shell
//git pull

//git pull = git fetch + git merge
git pull <远程主机名> <远程分支名>:<本地分支名>
git pull				//远程分支与当前分支拉取并合并
git pull origin			
//git pull会将本地更新覆盖掉，使用时候注意git pull 和 git fetch的使用情况

git pull origin master:brantest	//将远程主机 origin 的 master 分支拉取过来，与本地的 brantest 分支合并
git pull origin master			//远程分支是与当前分支合并，则冒号后面的部分可以省略
```







## 同步远程分支代码到本地的命令

```shell
//git pull
git pull origin master							//获取下来直接自动合并，不安全

//git fetch
git fetch orgin master							//单独拉取远程分支代码

git log -p master..origin/master				//比较差异

git meger origin/master							//进行合并
```

> 注意：直接在某个分支下使用git push会有如下提示
> 执行一下：git push --set-upstream origin dev



## git冲突

没有设置远程的push分支git冲突。团队中两人同时fetch了同一个分支，第一个人修改后提交，第二个人提交就失败。
注意点：每次push之前，先pull或者fetch一下，保证您的分支代码始终是最新的。

解决方案：

```shell
//1.获取远程分支更新，也就是第一个人提交的
git fetch origin dev

//2.尝试由git带来的自动合并，如果两个分支的内容有差异，则提示合并失败
git merge origin/master 				//将origin/master合并到当前分支

//3.查看当前的状态，寻找帮助信息：
git status

//4.手动合并
git mergetool 文件名
```





### error: unable to create file xxx: Filename too long

在该 Clone 的路径下，运行该命令：

```
git config --global core.longpaths true
```

删除老的 Clone 项目，重新 Clone 即可！











场景：

拉取一个repository，在test分支基础上，本地新建一个dev分支，并远程新建dev分支，推送本地代码

```shell
git clone .......				//克隆代码
git checkout origin/test		//切换到远程的test分支
git chechout -b dev				//本地新建一个dev分支，并切换到本地dev
git push origin dev				//远程创建分支dev，并把当前dev分支推送到origin/dev分支上
```



1.查看最后一次提交记录的修改文件信息

 git show --raw

2.查看指定commit id对应修改文件列表

git show --raw commit_id

git show --raw 2f80f1c8bb2cb8e91d22ad38480b681c194f6518

3.查看所有提交记录的修改文件信息

git log --stat

git log --name-only

4.查看所有修改相关的commit ID和comment信息

git log --pretty=oneline

5.查询指定author的修改信息

git log --author=jack.li

6.查看指定author在指定时间修改信息

$ git log --pretty="%h - %s" --author='Junio C Hamano' --since="2008-10-01" \
   --before="2008-11-01" --no-merges -- t/
5610e3b - Fix testcase failure when extended attributes are in use
acd3b9e - Enhance hold_lock_file_for_{update,append}() API
f563754 - demonstrate breakage of detached checkout with symbolic link HEAD
d1a43f2 - reset --hard/read-tree --reset -u: remove unmerged new paths
51a94af - Fix "checkout --track -b newbranch" on detached HEAD
b0ad11e - pull: allow "git pull origin $something:$current_branch" into an unborn branch
