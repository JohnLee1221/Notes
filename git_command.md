# git command

## git init

## git log

## git status

## git add .

## git commit -m "xxxx"



## git checkout <file_name>

丢弃工作区的修改，并用最近一次的 [commit](https://so.csdn.net/so/search?q=commit&spm=1001.2101.3001.7020) 内容还原到当前工作区（还原对文件中内容的操作，无法对添加文件、删除文件起作用）



## git reset --hard "commit_ID"

清空暂存区，将已提交的内容的版本恢复到本地，本地的文件也将被恢复的版本替换（恢复到上一次 commit 后的状态，上一次 commit 后的修改也丢弃）



## git clean -xdf



## git checkout -b "123"



## git push --set-upstream origin "123"
