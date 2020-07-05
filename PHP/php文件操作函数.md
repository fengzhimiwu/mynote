fopen() 打开文件或者 URL
fsockopen（）打开一个网络连接或者一个Unix套接字连接
fwrite () 写入文件
basename() 返回路径中的文件名部分。
copy()复制文件
dirname () 返回路径中的目录部分
disk_free_space () 返回目录中的可用空间
disk_total_space () 返回一个目录的磁盘总大小
fclose 关闭一个已打开的文件指针
feof() 测试文件指针是否到了文件结束的位置
fgetc 从文件指针中读取一个字符
fgets 从文件指针中读取一行
fgetss() 从打开的文件中读取一行并过滤掉 HTML 和 PHP 标记。
file() 把整个文件读入一个数组中
file_exists ( string $filename ) : bool 检查文件或目录是否存在
file_get_contents () 将整个文件读入一个字符串
file_put_contents 将一个字符串写入文件
filesize() 函数返回指定文件的大小 ,返回字节数
filetype() 返回指定文件或目录的类型。
flock() 锁定或释放文件
unlink() 删除文件
fgetcsv() 从文件指针中读入一行并解析 CSV 字段
fputcsv() 将行格式化为 CSV 并写入文件指针
fpassthru() 函数输出文件指针处的所有剩余数据
parse_ini_file() 函数解析一个配置文件，并以数组的形式返回其中的设置。
pathinfo() 以数组的形式返回文件路径的信息
realpath() 返回文件的绝对路径
rename() 函数重命名文件或目录
mkdir() 创建目录
move_uploaded_file() 将上传的文件移动到新位置
rmdir() 删除空目录
touch() 设置指定文件的访问和修改时间
unlink() 删除文件
fseek() 在打开的文件中定位
rewind() 将文件指针的位置倒回文件的开头。
glob() 寻找与模式匹配的文件路径
scandir() 列出指定路径中的文件和目录
opendir() 打开目录
readdir() 获取打开目录中的一条子目录/文件名称
closedir(); 关闭目录
is_dir() 判断指定的文件名是否是一个目录。
is_executable() 判断文件是否可执行。
is_file() 判断指定文件是否为常规的文件。
is_link() 判断指定的文件是否是连接。
is_readable() 判断文件是否可读。
is_uploaded_file() 判断文件是否是通过 HTTP POST 上传的。
is_writable() 判断文件是否可写。
统计指定文件夹下的所有文件跟目录数

```php
$numdir = $numfile = 0;
function foo ($dir)
{
    global $numdir,$numfile;
    foreach (scandir($dir) as $key => $val)
    {
        if ($val !== '.' && $val !== '..')
        {
            if ( is_dir($file = $dir.'/'.$val))
            {
                ++$numdir;//目录数
                foo($file);
            }
            if ( is_file($dir.'/'.$val))
            {
                ++$numfile;//文件数量
            }
        }
    }
}
foo('laravel');
echo '目录数：'.$numdir.PHP_EOL;
echo '文件数：'.$numfile;
```



### 文件函数

- [basename](https://www.php.net/manual/zh/function.basename.php) — 返回路径中的文件名部分
- [chgrp](https://www.php.net/manual/zh/function.chgrp.php) — 改变文件所属的组
- [chmod](https://www.php.net/manual/zh/function.chmod.php) — 改变文件模式
- [chown](https://www.php.net/manual/zh/function.chown.php) — 改变文件的所有者
- [clearstatcache](https://www.php.net/manual/zh/function.clearstatcache.php) — 清除文件状态缓存
- [copy](https://www.php.net/manual/zh/function.copy.php) — 拷贝文件
- [delete](https://www.php.net/manual/zh/function.delete.php) — 参见 unlink 或 unset
- [dirname](https://www.php.net/manual/zh/function.dirname.php) — 返回路径中的目录部分
- [disk_free_space](https://www.php.net/manual/zh/function.disk-free-space.php) — 返回目录中的可用空间
- [disk_total_space](https://www.php.net/manual/zh/function.disk-total-space.php) — 返回一个目录的磁盘总大小
- [diskfreespace](https://www.php.net/manual/zh/function.diskfreespace.php) — disk_free_space 的别名
- [fclose](https://www.php.net/manual/zh/function.fclose.php) — 关闭一个已打开的文件指针
- [feof](https://www.php.net/manual/zh/function.feof.php) — 测试文件指针是否到了文件结束的位置
- [fflush](https://www.php.net/manual/zh/function.fflush.php) — 将缓冲内容输出到文件
- [fgetc](https://www.php.net/manual/zh/function.fgetc.php) — 从文件指针中读取字符
- [fgetcsv](https://www.php.net/manual/zh/function.fgetcsv.php) — 从文件指针中读入一行并解析 CSV 字段
- [fgets](https://www.php.net/manual/zh/function.fgets.php) — 从文件指针中读取一行
- [fgetss](https://www.php.net/manual/zh/function.fgetss.php) — 从文件指针中读取一行并过滤掉 HTML 标记
- [file_exists](https://www.php.net/manual/zh/function.file-exists.php) — 检查文件或目录是否存在
- [file_get_contents](https://www.php.net/manual/zh/function.file-get-contents.php) — 将整个文件读入一个字符串
- [file_put_contents](https://www.php.net/manual/zh/function.file-put-contents.php) — 将一个字符串写入文件
- [file](https://www.php.net/manual/zh/function.file.php) — 把整个文件读入一个数组中
- [fileatime](https://www.php.net/manual/zh/function.fileatime.php) — 取得文件的上次访问时间
- [filectime](https://www.php.net/manual/zh/function.filectime.php) — 取得文件的 inode 修改时间
- [filegroup](https://www.php.net/manual/zh/function.filegroup.php) — 取得文件的组
- [fileinode](https://www.php.net/manual/zh/function.fileinode.php) — 取得文件的 inode
- [filemtime](https://www.php.net/manual/zh/function.filemtime.php) — 取得文件修改时间
- [fileowner](https://www.php.net/manual/zh/function.fileowner.php) — 取得文件的所有者
- [fileperms](https://www.php.net/manual/zh/function.fileperms.php) — 取得文件的权限
- [filesize](https://www.php.net/manual/zh/function.filesize.php) — 取得文件大小
- [filetype](https://www.php.net/manual/zh/function.filetype.php) — 取得文件类型
- [flock](https://www.php.net/manual/zh/function.flock.php) — 轻便的咨询文件锁定
- [fnmatch](https://www.php.net/manual/zh/function.fnmatch.php) — 用模式匹配文件名
- [fopen](https://www.php.net/manual/zh/function.fopen.php) — 打开文件或者 URL
- [fpassthru](https://www.php.net/manual/zh/function.fpassthru.php) — 输出文件指针处的所有剩余数据
- [fputcsv](https://www.php.net/manual/zh/function.fputcsv.php) — 将行格式化为 CSV 并写入文件指针
- [fputs](https://www.php.net/manual/zh/function.fputs.php) — fwrite 的别名
- [fread](https://www.php.net/manual/zh/function.fread.php) — 读取文件（可安全用于二进制文件）
- [fscanf](https://www.php.net/manual/zh/function.fscanf.php) — 从文件中格式化输入
- [fseek](https://www.php.net/manual/zh/function.fseek.php) — 在文件指针中定位
- [fstat](https://www.php.net/manual/zh/function.fstat.php) — 通过已打开的文件指针取得文件信息
- [ftell](https://www.php.net/manual/zh/function.ftell.php) — 返回文件指针读/写的位置
- [ftruncate](https://www.php.net/manual/zh/function.ftruncate.php) — 将文件截断到给定的长度
- [fwrite](https://www.php.net/manual/zh/function.fwrite.php) — 写入文件（可安全用于二进制文件）
- [glob](https://www.php.net/manual/zh/function.glob.php) — 寻找与模式匹配的文件路径
- [is_dir](https://www.php.net/manual/zh/function.is-dir.php) — 判断给定文件名是否是一个目录
- [is_executable](https://www.php.net/manual/zh/function.is-executable.php) — 判断给定文件名是否可执行
- [is_file](https://www.php.net/manual/zh/function.is-file.php) — 判断给定文件名是否为一个正常的文件
- [is_link](https://www.php.net/manual/zh/function.is-link.php) — 判断给定文件名是否为一个符号连接
- [is_readable](https://www.php.net/manual/zh/function.is-readable.php) — 判断给定文件名是否可读
- [is_uploaded_file](https://www.php.net/manual/zh/function.is-uploaded-file.php) — 判断文件是否是通过 HTTP POST 上传的
- [is_writable](https://www.php.net/manual/zh/function.is-writable.php) — 判断给定的文件名是否可写
- [is_writeable](https://www.php.net/manual/zh/function.is-writeable.php) — is_writable 的别名
- [lchgrp](https://www.php.net/manual/zh/function.lchgrp.php) — 修改符号链接的所有组
- [lchown](https://www.php.net/manual/zh/function.lchown.php) — 修改符号链接的所有者
- [link](https://www.php.net/manual/zh/function.link.php) — 建立一个硬连接
- [linkinfo](https://www.php.net/manual/zh/function.linkinfo.php) — 获取一个连接的信息
- [lstat](https://www.php.net/manual/zh/function.lstat.php) — 给出一个文件或符号连接的信息
- [mkdir](https://www.php.net/manual/zh/function.mkdir.php) — 新建目录
- [move_uploaded_file](https://www.php.net/manual/zh/function.move-uploaded-file.php) — 将上传的文件移动到新位置
- [parse_ini_file](https://www.php.net/manual/zh/function.parse-ini-file.php) — 解析一个配置文件
- [parse_ini_string](https://www.php.net/manual/zh/function.parse-ini-string.php) — 解析配置字符串
- [pathinfo](https://www.php.net/manual/zh/function.pathinfo.php) — 返回文件路径的信息
- [pclose](https://www.php.net/manual/zh/function.pclose.php) — 关闭进程文件指针
- [popen](https://www.php.net/manual/zh/function.popen.php) — 打开进程文件指针
- [readfile](https://www.php.net/manual/zh/function.readfile.php) — 输出文件
- [readlink](https://www.php.net/manual/zh/function.readlink.php) — 返回符号连接指向的目标
- [realpath_cache_get](https://www.php.net/manual/zh/function.realpath-cache-get.php) — 获取真实目录缓存的详情
- [realpath_cache_size](https://www.php.net/manual/zh/function.realpath-cache-size.php) — 获取真实路径缓冲区的大小
- [realpath](https://www.php.net/manual/zh/function.realpath.php) — 返回规范化的绝对路径名
- [rename](https://www.php.net/manual/zh/function.rename.php) — 重命名一个文件或目录
- [rewind](https://www.php.net/manual/zh/function.rewind.php) — 倒回文件指针的位置
- [rmdir](https://www.php.net/manual/zh/function.rmdir.php) — 删除目录
- [set_file_buffer](https://www.php.net/manual/zh/function.set-file-buffer.php) — stream_set_write_buffer 的别名
- [stat](https://www.php.net/manual/zh/function.stat.php) — 给出文件的信息
- [symlink](https://www.php.net/manual/zh/function.symlink.php) — 建立符号连接
- [tempnam](https://www.php.net/manual/zh/function.tempnam.php) — 建立一个具有唯一文件名的文件
- [tmpfile](https://www.php.net/manual/zh/function.tmpfile.php) — 建立一个临时文件
- [touch](https://www.php.net/manual/zh/function.touch.php) — 设定文件的访问和修改时间
- [umask](https://www.php.net/manual/zh/function.umask.php) — 改变当前的 umask
- [unlink](https://www.php.net/manual/zh/function.unlink.php) — 删除文件



### 目录函数

- [chdir](https://www.php.net/manual/zh/function.chdir.php) — 改变目录
- [chroot](https://www.php.net/manual/zh/function.chroot.php) — 改变根目录
- [closedir](https://www.php.net/manual/zh/function.closedir.php) — 关闭目录句柄
- [dir](https://www.php.net/manual/zh/function.dir.php) — 返回一个 Directory 类实例
- [getcwd](https://www.php.net/manual/zh/function.getcwd.php) — 取得当前工作目录
- [opendir](https://www.php.net/manual/zh/function.opendir.php) — 打开目录句柄
- [readdir](https://www.php.net/manual/zh/function.readdir.php) — 从目录句柄中读取条目
- [rewinddir](https://www.php.net/manual/zh/function.rewinddir.php) — 倒回目录句柄
- [scandir](https://www.php.net/manual/zh/function.scandir.php) — 列出指定路径中的文件和目录