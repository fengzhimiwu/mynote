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
print(sum(2, '3', 4.1)); // 结果：9

// 严格模式
declare(strict_types=1);
function sum(int ...$ints)
{
   return array_sum($ints);
}
print(sum(2, '3', 4.1));
// 结果：PHP Fatal error:  Uncaught TypeError: Argument 2 passed to sum() mu...
```

### 返回类型声明

```php
declare(strict_types=1);
function returnIntValue(int $value): int
{
   return $value;
}
print(returnIntValue(5)); // 结果：5


declare(strict_types=1);
function returnIntValue(int $value): int
{
   return $value + 1.0;
}
print(returnIntValue(5));
// 结果：Fatal error: Uncaught TypeError: Return value of returnIntValue() ...
```

### void 函数

一个新的返回值类型void被引入。 返回值声明为 void 类型的方法要么干脆省去 return 语句，要么使用一个空的 return 语句。 对于 void 函数来说，NULL 不是一个合法的返回值。

返回的类型还有 void，定义返回类型为 void 的函数不能有返回值，即使返回 null 也不行。

void 函数可以省去 return 语句，或者使用一个空的 return 语句。

### NULL 合并运算符

```
$site = isset($_GET['site']) ? $_GET['site'] : 'default';

$site = $_GET['site'] ?? 'default';
```

### 太空船运算符（组合比较符）

新增加的太空船运算符（组合比较符）用于比较两个表达式 **$a** 和 **$b**，如果 **$a** 小于、等于或大于 **$b**时，它分别返回-1、0或1。

1 <=> 2        1 <=> 1     2 <=> 1

### 常量数组

在 PHP 5.6 中仅能通过 const 定义常量数组，PHP 7 可以通过 define() 来定义。

```
// 使用 define 函数来定义数组
define('sites', [
   '123',
   '456',
   '789'
]);
```

### 匿名类

支持通过 **new class** 来实例化一个匿名类，这可以用来替代一些"用后即焚"的完整类定义。

```php
$app = new Application;
// 使用 new class 创建匿名类
$app->setLogger(new class implements Logger {
   public function log(string $msg) {
      print($msg);
   }
});
$app->getLogger()->log("我的第一条日志");
```

### Closure::call()

PHP 7 的 Closure::call() 有着更好的性能，将一个闭包函数动态绑定到一个新的对象实例并调用执行该函数。

```php
class A {
    private $x = 1;
}

// PHP 7 之前版本定义闭包函数代码
$getXCB = function() {
    return $this->x;
};

// 闭包函数绑定到类 A 上
$getX = $getXCB->bindTo(new A, 'A'); 

echo $getX();
print(PHP_EOL);

// PHP 7+ 代码
$getX = function() {
    return $this->x;
};
echo $getX->call(new A);
```

### 过滤 unserialize()

增加了可以为 unserialize() 提供过滤的特性，可以防止非法数据进行代码注入，提供了更安全的反序列化数据。

```php
class MyClass1 { 
   public $obj1prop;   
}
class MyClass2 {
   public $obj2prop;
}


$obj1 = new MyClass1();
$obj1->obj1prop = 1;
$obj2 = new MyClass2();
$obj2->obj2prop = 2;

$serializedObj1 = serialize($obj1);
$serializedObj2 = serialize($obj2);

// 默认行为是接收所有类
// 第二个参数可以忽略
// 如果 allowed_classes 设置为 false, unserialize 会将所有对象转换为 __PHP_Incomplete_Class 对象
$data = unserialize($serializedObj1 , ["allowed_classes" => true]);

// 转换所有对象到 __PHP_Incomplete_Class 对象，只允许 MyClass1 和 MyClass2 转换到 __PHP_Incomplete_Class
$data2 = unserialize($serializedObj2 , ["allowed_classes" => ["MyClass1", "MyClass2"]]);

print($data->obj1prop);
print(PHP_EOL);
print($data2->obj2prop);
```

### IntlChar()

PHP 7 通过 intl 扩展来支持国际化 (i18n) 和本地化 (l10n) 。此扩展仅仅是对 ICU 库的基础包装，并提供了和 ICU 库类似的方法和特性。

PHP 7 通过新的 IntlChar 类暴露出 ICU 中的 Unicode 字符特性。这个类自身定义了许多静态方法用于操作多字符集的 unicode 字符。

```php
printf('%x', IntlChar::CODEPOINT_MAX);
echo IntlChar::charName('@');
var_dump(IntlChar::ispunct('!'));
//10ffff
//COMMERCIAL AT
//bool(true)
```

### CSPRNG

CSPRNG（Cryptographically Secure Pseudo-Random Number Generator，伪随机数产生器）。

PHP 7 通过引入几个 CSPRNG 函数提供一种简单的机制来生成密码学上强壮的随机数。

- **random_bytes()** - 加密生存被保护的伪随机字符串。
- **random_int()** - 加密生存被保护的伪随机整数。

#### random_bytes()

**语法格式**

```
string random_bytes ( int $length )
```

参数：**length** - 随机字符串返回的字节数。

返回值：返回一个字符串，接受一个int型入参代表返回结果的字节数。

```php
$bytes = random_bytes(5);
print(bin2hex($bytes)); // 6f36d48a29
```

#### random_int()

**语法格式**

```
int random_int ( int $min , int $max )
```

**参数**

- **min** - 返回的最小值，必须是大于或等于 PHP_INT_MIN 。
- **max** - 返回的最大值，必须是小于或等于 PHP_INT_MAX 。

**返回值**

- 返回一个指定范围内的int型数字。

```php
print(random_int(100, 999));
print(PHP_EOL);
print(random_int(-1000, 0));
//723
//-64
```

### use 语句

可以使用一个 use 从同一个 namespace 中导入类、函数和常量：

```php
// PHP 7+ 版本可以使用一个 use 导入同一个 namespace 的类
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
```

### intdiv() 函数

接收两个参数，返回值为第一个参数除于第二个参数的值并取整。



参考链接：https://www.runoob.com/php/php7-new-features.html