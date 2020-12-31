在工程的根目录下composer.json所定义的包称为root包。下面是对Composer配置文件composer.json中的命令的初步解释。

1.require
   格式为： "require":{"vendor-name/package-name":"version", ...}
   名字部分会作为vendor下的路径进行创建
   版本支持精确的版本号，也支持范围如>=1.0; >=1.0,<2.0; ","作为逻辑与，而"!"作为逻辑或的意思。示例中使用了通配符*
   版本也支持tag或branch名称。
   类似的有require-dev，前者用于声明项目发布版本的依赖包，后者用于声明项目开发或测试中依赖的包。
2.autoload
   composer支持PSR-0,PSR-4,classmap及files包含以支持文件自动加载。PSR-4为推荐方式。
   2.1 Files类型
   格式："autoload":{"files":["path/to/1.php","path/to/2.php",...]}
   支持将数组中的文件进行自动加载，文件的路径相对于项目的根目录。缺点是麻烦，需要将所有文件都写进配置。
   2.2 classmap类型
   格式："autoload":{"classmap": ["path/to/src1","path/to/src2",...]}
   支持将数组中的路径下的文件进行自动加载。其很方便，但缺点是一旦增加了新文件，需要执行dump-autoload命令重新生成映射  文件vendor/composer/autoload_classmap.php。
   2.3 psr-0类型
   格式："autoload":

                             {
    
                                    "psr-0":
    
                                              {
    
                                                       "name1\\space\\":["path/",...],
                                                        "name2\\space\\":["path2/",...],
                                               }
                              }
    支持将命名空间映射到路径。命名空间结尾的\\不可省略。当执行install或update时，加载信息会写入vendor/composer/autoload_namespace.php文件。如果希望解析指定路径下的所有命名空间，则将命名空间置为空串即可。
    需要注意的是对应name2\space\Foo类的类文件的路径为path2/name2/space/Foo.php
    2.4 psr-4类型
    格式："autoload":
    
                               {
    
                                       "psr-4":
    
                                                 {
    
                                                          "name1\\space\\":["path/",...],
                                                          "name2\\space\\":["path2/",...],
                                                 }
                                }
     支持将命名空间映射到路径。命名空间结尾的\\不可省略。当执行install或update时，加载信息会写入vendor/composer/autoload_psr4.php文件。如果希望解析指定路径下的所有命名空间，则将命名空间置为空串即可。
     需要注意的是对应name2\space\Foo类的类文件的路径为path2/space/Foo.php，name2不出现在路径中。
     PSR-4和PSR-0最大的区别是对下划线（underscore)的定义不同。PSR-4中，在类名中使用下划线没有任何特殊含义。而PSR-0则规定类名中的下划线_会被转化成目录分隔符。
3.name
格式："name":"vendor/package"
如果要发布一个包，你需要指定包的名字信息。
4.version
格式："version":"1.0.2"
如果要发布一个包，你需要指定包的版本号。版本号的格式为X.Y.Z或vX.Y.Z，其后可以加后缀如-dev,-patch,-alpha,-beta或-RC。除dev外，尾上还可加一个数字，如1.0.0-alpha3。
5.description
格式："description":"your own description at here!"
如果要发布一个包，可以指定一个简短的介绍
6.type
格式："type":"library"
说明包的类型，支持如下library,project,metapackage,composer-plugin，默认为library
7.keywords
格式："keywords":["logging","database","redis"]
一个数组的关键字，用于搜索或过滤时使用。
8.homepage
可选的，说明项目的网站地址
9.time/license
说明项目的时间和License，时间格式为YY-MM-DD HH:MM:SS
10.authors
格式："authors":[
{"name":"ss","email":"ss@ss.com","homepage":"","role":""},...
]
用于说明项目的作者信息，为可选的。
11.support
格式："support":{"emial":"","issues":"","forum":"","wiki":"","irc":"" }
用于说明项目的支持信息
12.conflict
用于声明与本包有冲突的包的版本，使用类似于require。
13.replace
用于声明需要替换的包，使用类似于require
14.provided
用于说明本包实现了某个包的接口
15.suggest
格式："suggest":{"vendor/package":"Some description!"}
用于说明可选的，用于增强功能的包及说明。