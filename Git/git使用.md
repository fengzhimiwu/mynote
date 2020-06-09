
```
git init
git add readme.txt
git diff //查看修改
git status //查看状态
git commint -m "test"
git status //查看状态
git log --pretty=oneline
git reset --hard HEAD^ //回退版本(可以根据版本号)
git reflog //所有历史版本
git checkout -- readme.txt //撤销修改
```

场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。


```
git checkout -b dev //创建dev 分支
```
git checkout命令加上-b参数表示创建并切换，相当于以下两条命令：

```
$ git branch dev
$ git checkout dev
```


```
git branch //查看当前分支
git checkout dev //切换dev分支
git merge dev //将dev切换到master
```

Git鼓励大量使用分支：

查看分支：git branch

创建分支：git branch <name>

切换分支：git checkout <name>

创建+切换分支：git checkout -b <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>


```
git merge --no-ff -m "merge with no-ff" dev//禁用快速合并
```

当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug，修复后，再git stash pop，回到工作现场。

开发一个新feature，最好新建一个分支；

如果要丢弃一个没有被合并过的分支，可以通过git branch -D <name>强行删除


```
git push origin dev //推送到dev分支
```
master分支是主分支，因此要时刻与远程同步；

dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；

bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；

feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

因此，多人协作的工作模式通常是这样：

首先，可以试图用git push origin branch-name推送自己的修改；

如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

如果合并有冲突，则解决冲突，并在本地提交；

没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！

如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream branch-name origin/branch-name。

查看远程库信息，使用git remote -v；

本地新建的分支如果不推送到远程，对其他人就是不可见的；

从本地推送分支，使用git push origin branch-name，如果推送失败，先用git pull抓取远程的新提交；

在本地创建和远程分支对应的分支，使用git checkout -b branch-name origin/branch-name，本地和远程分支的名称最好一致；

建立本地分支和远程分支的关联，使用git branch --set-upstream branch-name origin/branch-name；

从远程抓取分支，使用git pull，如果有冲突，要先处理冲突。

命令git tag <name>用于新建一个标签，默认为HEAD，也可以指定一个commit id；

git tag -a <tagname> -m "blablabla..."可以指定标签信息；

git tag -s <tagname> -m "blablabla..."可以用PGP签名标签；

命令git tag可以查看所有标签。


命令git push origin <tagname>可以推送一个本地标签；

命令git push origin --tags可以推送全部未推送过的本地标签；

命令git tag -d <tagname>可以删除一个本地标签；

命令git push origin :refs/tags/<tagname>可以删除一个远程标签。

我们可以删除已有的GitHub远程库：

git remote rm origin
再关联码云的远程库（注意路径中需要填写正确的用户名）：

git remote add origin git@gitee.com:liaoxuefeng/learngit.git



#### Git 中 warning: LF will be replaced by CRLF in readme.txt.问题解决

意思是：
警告:LF将被readme.txt中的CRLF替换。该文件将在工作目录中以其原始行结尾。
出现此问题是因为不同操作系统的使用的换行符不同：
Linux / Unix 采用换行符LF表示下一行
Windows  采用回车+换行 CRLF表示下一行
解决：可以通过设置 core.autocrlf 的值解决
$ git config --global core.autocrlf false    # 关闭自动转换 

