# git备忘录

1. gitconfig文件： 命令简写
* 位置：/Users/username
* gitignore文件： 全局文件
* .git/config 文件可查看仓库位置信息
* 单独设置一个仓库的email:当前仓库内 git config user.email "ccking@foxmail.com"  
  或者在 bash_profile 文件中加入：  
  `alias github='git config user.email "ccking@foxmail.com"'`  
  最后编译bash文件  source .bash_profile
* cat .git/logs/HEAD  查看仓库的log信息
* git fetch 和 git pull   
  * 都是从远程分支获取最新的版本到本地
  * git fetch 不会自动 merge
  
  		git fetch origin master
  		git log -p master..origin/master
  		git merge origin/master
  	1、从远程origin的master分支下载最新版本到 origin/master 分支  
  	2、比较本地master分支和远程master分支的区别  
  	3、合并代码  
  	上述过程的描述:
  	
  		git fetch origin master:tmp
  		git diff tmp
  		git merge tmp
  * git pull 也是从远程master分支获取最新版本但是会自动merge
  * git fetch 相对安全点

* 分支操作
  * 显示本地远程所有分支 git br -a 
  * 删除本地分支 git br -d dev/0.1.2
  * 删除远程分支 git push origin :dev/0.2.9
  * git br -m old new 重命名分支
  * git br -v 查看每一个分至最后一次commit
  * git co -  快速切换到上一个分支
  * git co -b test dev 基于dev新建test分支，并切换
  	
* 一个自动提交的shell脚本  
  * 一个命令全部提交，但是，不推荐使用
  * 地址：[autoPush.sh](https://github.com/ccforward/cc/blob/master/git/autoPush.sh)