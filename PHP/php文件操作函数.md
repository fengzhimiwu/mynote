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

