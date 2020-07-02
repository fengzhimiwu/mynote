MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串(字符)类型。



**数值类型**

<table class="reference" style="height: 282px; width: 1108px;">
<tbody>
<tr style="height: 5%;" align="center" valign="middle"><th style="background-color: #008b8b;"><span style="font-size: 16px; color: #ffffff; font-family: 'Microsoft YaHei';"><strong>类型</strong></span></th><th style="background-color: #008b8b;"><span style="font-size: 16px; color: #ffffff; font-family: 'Microsoft YaHei';"><strong>大小</strong></span></th><th style="background-color: #008b8b;"><span style="font-size: 16px; color: #ffffff; font-family: 'Microsoft YaHei';"><strong>范围（有符号）</strong></span></th><th style="background-color: #008b8b;"><span style="font-size: 16px; color: #ffffff; font-family: 'Microsoft YaHei';"><strong>范围（无符号）</strong></span></th><th style="background-color: #008b8b;"><span style="font-size: 16px; color: #ffffff; font-family: 'Microsoft YaHei';"><strong>用途</strong></span></th></tr>
<tr style="height: 5%; margin-left: 30px; background-color: #f0f8ff;" align="center" valign="middle">
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">TINYINT</span></td>
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">1 字节</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(-128，127)</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(0，255)</span></td>
<td style="width: 1%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">小整数值</span></td>
</tr>
<tr style="height: 5%; margin-left: 30px;" align="center" valign="middle">
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">SMALLINT</span></td>
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">2 字节</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(-32 768，32 767)</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(0，65 535)</span></td>
<td style="width: 1%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">大整数值</span></td>
</tr>
<tr style="height: 5%; margin-left: 30px; background-color: #f0f8ff;" align="center" valign="middle">
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">MEDIUMINT</span></td>
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">3 字节</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(-8 388 608，8 388 607)</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(0，16 777 215)</span></td>
<td style="width: 1%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">大整数值</span></td>
</tr>
<tr style="height: 5%; margin-left: 30px;" align="center" valign="middle">
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">INT或INTEGER</span></td>
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">4 字节</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(-2 147 483 648，2 147 483 647)</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(0，4 294 967 295)</span></td>
<td style="width: 1%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">大整数值</span></td>
</tr>
<tr style="height: 5%; margin-left: 30px; background-color: #f0f8ff;" align="center" valign="middle">
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">BIGINT</span></td>
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">8 字节</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(-9 233 372 036 854 775 808，9 223 372 036 854 775 807)</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(0，18 446 744 073 709 551 615)</span></td>
<td style="width: 1%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">极大整数值</span></td>
</tr>
<tr style="height: 5%; margin-left: 30px;" align="center" valign="middle">
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">FLOAT</span></td>
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">4 字节</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38)</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0，(1.175 494 351 E-38，3.402 823 466 E+38)</span></td>
<td style="width: 1%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">单精度</span><br><span style="font-size: 14px; font-family: 'Microsoft YaHei';">浮点数值</span></td>
</tr>
<tr style="height: 5%; margin-left: 30px; background-color: #f0f8ff;" align="center" valign="middle">
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">DOUBLE</span></td>
<td style="width: 1%; margin-left: 30px; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">8 字节</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">(-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)</span></td>
<td style="width: 5%; margin-left: 30px; text-align: left;" scope="colgroup" align="center" valign="middle"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)</span></td>
<td style="width: 1%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">双精度</span><br><span style="font-size: 14px; font-family: 'Microsoft YaHei';">浮点数值</span></td>
</tr>
</tbody>
</table>

**字符串**

字符串类型指CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET。该节描述了这些类型如何工作以及如何在查询中使用这些类型。

char和varchar：

1.char(n) 若存入字符数小于n，则以空格补于其后，查询之时再将空格去掉。所以char类型存储的字符串末尾不能有空格，varchar不限于此。 
2.char(n) 固定长度，char(4)不管是存入几个字符，都将占用4个字节，varchar是存入的实际字符数+1个字节（n<=255）或2个字节(n>255)，所以varchar(4),存入3个字符将占用4个字节。 
3.char类型的字符串检索速度要比varchar类型的快。



varchar和text： 

1.varchar可指定n，text不能指定，内部存储varchar是存入的实际字符数+1个字节（n<=255）或2个字节(n>255)，text是实际字符数+2个字节。 
2.text类型不能有默认值。 
3.varchar可直接创建索引，text创建索引要指定前多少个字符。varchar查询速度快于text,在都创建索引的情况下，text的索引似乎不起作用。

<table class="reference">
<thead>
<tr style="background-color: #008b8b;">
<td style="width: 20%; height: 20%;"><span style="font-size: 16px; color: #ffffff; font-family: 'Microsoft YaHei';">类型</span></td>
<th width="25%"><span style="font-size: 16px; color: #ffffff; font-family: 'Microsoft YaHei';">大小</span></th><th width="55%"><span style="font-size: 16px; color: #ffffff; font-family: 'Microsoft YaHei';">用途</span></th></tr>
</thead>
<tbody>
<tr style="height: 5%; background-color: #f0f8ff;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">CHAR</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-255字节</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">定长字符串</span></td>
</tr>
<tr style="height: 5%;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">VARCHAR</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-65535 字节</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">变长字符串</span></td>
</tr>
<tr style="height: 5%; background-color: #f0f8ff;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">TINYBLOB</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-255字节</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">不超过 255 个字符的二进制字符串</span></td>
</tr>
<tr style="height: 5%;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">TINYTEXT</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-255字节</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">短文本字符串</span></td>
</tr>
<tr style="height: 5%; background-color: #f0f8ff;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">BLOB</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-65 535字节</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">二进制形式的长文本数据</span></td>
</tr>
<tr style="height: 5%;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">TEXT</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-65 535字节</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">长文本数据</span></td>
</tr>
<tr style="height: 5%; background-color: #f0f8ff;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">MEDIUMBLOB</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-16 777 215字节</span></td>
<td style="height: 5%; text-align: left;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">二进制形式的中等长度文本数据</span></td>
</tr>
<tr style="height: 5%;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">MEDIUMTEXT</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-16 777 215字节</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">中等长度文本数据</span></td>
</tr>
<tr style="height: 5%; background-color: #f0f8ff;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">LONGBLOB</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-4 294 967 295字节</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">二进制形式的极大文本数据</span></td>
</tr>
<tr style="height: 5%;">
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">LONGTEXT</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">0-4 294 967 295字节</span></td>
<td style="height: 5%;"><span style="font-size: 14px; font-family: 'Microsoft YaHei';">极大文本数据</span></td>
</tr>
</tbody>
</table>

**日期时间类型**

表示时间值的日期和时间类型为DATETIME、DATE、TIMESTAMP、TIME和YEAR。

| 类型      | 大小  | 范围                                                         | 格式                | 用途                     |
| --------- | ----- | ------------------------------------------------------------ | ------------------- | ------------------------ |
| DATE      | 3字节 | 1000-01-01/9999-12-31                                        | YYYY-MM-DD          | 日期值                   |
| TIME      | 3字节 | '-838:59:59'/'838:59:59'                                     | HH:MM:SS            | 时间值或持续时间         |
| YEAR      | 1字节 | 1901/2155                                                    | YYYY                | 年份值                   |
| DATETIME  | 8字节 | 1000-01-01 00:00:00/9999-12-31 23:59:59                      | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
| TIMESTAMP | 4字节 | 1970-01-01<br />00:00:00/2038<br />结束时间是<br />第 2147483647 秒，<br />北京时间 2038-1-19 11:14:07，<br />格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |



**二进制** 

布尔:bit
bit 表示1个二进制的位
bit(8) 表示8个二进制的位
性别可以定义为0,1, 而不使用male或female字符串
数据逻辑删除
某辆车在车库中停放的状态
所有基于两种状态的数据都可以使用0,1来存储.



原文链接：https://www.cnblogs.com/jennyyin/p/7895010.html