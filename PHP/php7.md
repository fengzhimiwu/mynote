区别


```
$a = $c ?? $b;  等同于   $a = isset($c) ? $c : $b;

$a = $c ?: $b;  等同于   $a = $c ? $c : $b;
```

### 标量类型声明

默认情况下，所有的PHP文件都处于弱类型校验模式。

PHP 7 增加了标量类型声明的特性，标量类型声明有两种模式:

- 强制模式 (默认)
- 严格模式

标量类型声明语法格式：

```php
declare(strict_types=1); 
```

代码中通过指定 strict_types的值（1或者0），1表示严格类型校验模式，作用于函数调用和返回语句；0表示弱类型校验模式。

可以使用的类型参数有：

int  float  bool  string  interfaces  array  callable

```php
// 强制模式
function sum(int ...$ints)
{
   return array_sum($ints);
}
print(sum(2, '3', 4.1));
```

以上程序执行输出结果为：

9

```php
// 严格模式
declare(strict_types=1);
function sum(int ...$ints)
{
   return array_sum($ints);
}
print(sum(2, '3', 4.1));
```

执行输出结果为：

PHP Fatal error:  Uncaught TypeError: Argument 2 passed to sum() must be of the type integer, string given, called in

### 返回类型声明

可以声明的返回类型同标量类型一样