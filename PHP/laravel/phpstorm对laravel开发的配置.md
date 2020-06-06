### 使用composer创建laravel项目

```
./ 的位置写的是项目名，这里因为我之前创建了文件夹的名称，所以直接进入到文件夹，然后在文件夹目录中创建laravel项目结构
一定要保证该文件夹中为空，不能包含任何文件，注意我这里使用的是5.6版本的，5.4的版本配置起来可能存在一些问题
composer create-project --prefer-dist laravel/laravel ./ "5.6.*"
```

### 1.安装Laravel跟踪提示插件

步骤：File->settings->plugins->laravel-plugin

下载好后重启phpstorm，在打开 File->settings->Languages & Frameworks -> PHP -> Laravel -> 勾选两个选项 -> apply应用

### 2.安装Laravel代码提示

参照安装文档 https://packagist.org/packages/barryvdh/laravel-ide-helper

安装laravel-ide-helper

```
composer require --dev barryvdh/laravel-ide-helper
```

修改配置文件 app/Providers/AppServiceProvider.php文件中添加

```
public function register()
{
    if ($this->app->environment() !== 'production') {
        $this->app->register(\Barryvdh\LaravelIdeHelper\IdeHelperServiceProvider::class);
    }
    // ...
}
```

生成代码跟踪支持,以上的步骤配置完成后基本上就ok了！

```
php artisan ide-helper:generate
```

### 3.配置Laravel的artisan命令提示

步骤：file -> settings -> tools -> command line tool support

右侧 +  ->  Tool base on Symfony Console -> global  -> ok

alias: artisan -> path to script:选择artisan文件 -> ok

ctrl + shift + x 



其他 方法

```
composer require --dev "barryvdh/laravel-ide-helper:^2.4"
composer install
```

然后执行

```
php artisan ide-helper:generate
```

重启phpstorm