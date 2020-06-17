# git rm–r folder/file fatal:pathspec "" did not match any files

### 问题描述：

某年某月某日，在查看git库的时候，发现文件的分布和文件夹的名字是极其不合理的，所以移动和重命名了某些文件。

在删除(*git rm –r folder*)一个**空文件夹**的时候，出现错误：fatal:pathspec "folder Name" did not match any files,就是说，git没有找到相应的文件。但是这个文件夹明明是存在的，刚把里面的文件移到其他的文件夹里面，此时这个文件夹是空的。

### 问题推测：

某人喜欢推测，知道这是个不好的习惯，但木有办法。在现有的库中，新建文件夹（**空的**），*git st* 发现我的库中并没有需要添加的内容，只是当

我在空的文件夹内，*touch new file* 后，此时库才发生了改变。此时才显示需要我*add*文件。当我把t*ouch*的文件*git rm –f new file*删除后，空的文件夹依然存在，但是，此时git，就没有add的提示了。

我的分析：git阔能对我的空文件夹untracked，我在删除的时候，index找不到。

### 问题解决：

git的clean command：**git-clean - Remove untracked files from the working tree**

**git clean –fd untracked folder**

或者进入目录，delete

也可指定文件进行删除



## Git 过滤已上传的文件夹

1、删除所有文件追踪状态

git rm -r --cached xxx

2、新增.gitignore文件，并将要忽略的目录写入文件