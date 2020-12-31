
（1）原始社会里，几乎没有社会分工。需要斧子的人(调用者)只能自己去磨一把斧子(被调用者)。

（2）进入工业社会，工厂出现。斧子不再由普通人完成，而在工厂里被生产出来，此时需要斧子的人(调用者)找到工厂，购买斧子，无须关心斧子的制造过程。

（3）进入“按需分配”社会，需要斧子的人不需要找到工厂，坐在家里发出一个简单指令:需要斧子。斧子就自然出现在他面前。

第一种情况下，实例的调用者创建被调用的实例，必然要求被调用的类出现在调用者的代码里。无法实现二者之间的松耦合。

第二种情况下，调用者无须关心被调用者具体实现过程，只需要找到符合某种标准(接口)的实例，即可使用。此时调用的代码面向接口编程，可以让调用者和被调用者[解耦](https://www.baidu.com/s?wd=解耦&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，这也是工厂模式大量使用的原因。但调用者需要自己定位工厂，调用者与特定工厂耦合在一起。

第三种情况下，调用者无须自己定位工厂，程序运行到需要被调用者时，依赖注入容器自动提供被调用者实例。事实上，调用者和被调用者都处于依赖注入容器的管理下，二者之间的依赖关系由依赖注入容器提供。因此调用者与被调用者的耦合度进一步降低，这使得应用更加容易维护，这就是依赖注入所要达到的目的。

## 用php实现一个轻量的依赖注入容器

首先我们创建一个类，看起来是这样的：

```php
<?php   
class Di
{
    protected $_service = [];
    public function set($name, $definition)
    {
        $this->_service[$name] = $definition;
    }
    public function get($name)
    {
        if (isset($this->_service[$name])) {
            $definition = $this->service[$name];
        } else {
            throw new Exception("Service '" . name . "' wasn't found in the dependency injection container");
        }

        if (is_object($definition)) {
            $instance = call_user_func($definition);
        }

        return $instance;
    }
}
```

现在我们已经有了一个简单的类，包含一个属性和两个方法。假设我们现在有两个类，redisDB和cache，redisDB提供一个redis数据库的操作，cache负责缓存功能的实现并且依赖于redisDB。

```php
class redisDB
{
    protected $_di;

    protected $_options;

    public function __construct($options = null)
    {
        $this->_options = $options;
    }

    public function setDI($di)
    {
        $this->_di = $di;
    }

    public function find($key, $lifetime)
    {
        // code
    }

    public function save($key, $value, $lifetime)
    {
        // code
    }

    public function delete($key)
    {
        // code
    }
}
```

在这个类中我们简单实现了redis的查询、保存和删除。你可能会有疑问，另外一个方法setDi是做什么的。待我继续为你讲解。另一个类和当前这个类结构很像：

```php
class cache
{
    protected $_di;

    protected $_options;

    protected $_connect;

    public function __construct($options = null)
    {
        $this->_options = $options;
    }

    public function setDI($di)
    {
        $this->_di = $di;
    }

    protected function _connect()
    {
        $options = $this->_options;
        if (isset($options['connect'])) {
            $service = $options['connect'];
        } else {
            $service = 'redis';
        }

        return $this->_di->get($service);
    }

    public function get($key, $lifetime)
    {
        $connect = $this->_connect;
        if (!is_object($connect)) {
            $connect = $this->_connect()
            $this->_connect = $connect;
        }
        // code
        ...
        return $connect->find($key, $lifetime);
    }

    public function save($key, $value, $lifetime)
    {
        $connect = $this->_connect;
        if (!is_object($connect)) {
            $connect = $this->_connect()
            $this->_connect = $connect;
        }
        // code
        ...
        return $connect->save($key, $lifetime);
    }

    public function delete($key)
    {
        $connect = $this->_connect;
        if (!is_object($connect)) {
            $connect = $this->_connect()
            $this->_connect = $connect;
        }
        // code
        ...
        $connect->delete($key, $lifetime);
    }
}
```

现在我们就当已经实现了redisDB和cache这两个组件，具体的细节这里就先不做讨论了，来看看如何使用使用吧。首先需要将两个组件注入到容器中：

```php
<?php
    $di = new Di();
    $di->set('redis', function() {
         return new redisDB([
             'host' => '127.0.0.1',
             'port' => 6379
         ]);
    });
    $di->set('cache', function() use ($di) {
        $cache = new cache([
            'connect' => 'redis'
        ]);
        $cache->setDi($di);
        return $cache;
    });


    // 然后在任何你想使用cache的地方
    $cache = $di->get('cache');
    $cache->get('key'); // 获取缓存数据
    $cache->save('key', 'value', 'lifetime'); // 保存数据
    $cache->delete('key'); // 删除数据
```

到这里你可能会觉得这样以来反而有点繁琐了。cache和redisDB的结构如此之像，完全可以把redis写到cache中而没必要单独分离出来？但是你想过没有，有些数据及时性没那么高而且数量比较大，用redis有点不合适，mongodb是更好的选择；有些数据更新频率更慢，对查询速度也没要求，直接写入文件保存到硬盘可能更为合适；再或者，你的客户觉得redis运维难度有点大，让你给他换成memcache… 这就是为什么把它分离出来了。然后，继续改进代码：

```php
interface BackendInterface {
    public function find($key, $lifetime);
    public function save($key, $value, $lifetime);
    public function delete($key);
}

class redisDB implements BackendInterface
{
    public function find($key, $lifetime) { }
    public function save($key, $value, $lifetime) { }
    public function delete($key) { }
}

class mongoDB implements BackendInterface
{
    public function find($key, $lifetime) { }
    public function save($key, $value, $lifetime) { }
    public function delete($key) { }
}

class file implements BackendInterface
{
    public function find($key, $lifetime) { }
    public function save($key, $value, $lifetime) { }
    public function delete($key) { }
}

$di = new Di();
//  redis
$di->set('redis', function() {
     return new redisDB([
         'host' => '127.0.0.1',
         'port' => 6379
     ]);
});
// mongodb
$di->set('mongo', function() {
     return new mongoDB([
         'host' => '127.0.0.1',
         'port' => 12707
     ]);
});
// file
$di->set('file', function() {
     return new file([
         'path' => 'path'
     ]);
});
// save at redis
$di->set('fastCache', function() use ($di) {
     $cache = new cache([
         'connect' => 'redis'
     ]);
     $cache->setDi($di);
     return $cache;
});
// save at mongodb
$di->set('cache', function() use ($di) {
     $cache = new cache([
         'connect' => 'mongo'
     ]);
     $cache->setDi($di);
     return $cache;
});
// save at file
$di->set('slowCache', function() use ($di) {
     $cache = new cache([
         'connect' => 'file'
     ]);
     $cache->setDi($di);
     return $cache;
});

// 然后在任何你想使用cache的地方 
$cache = $di->get('cache');
```

我们新增加了一个接口BackendInterface，规定了redisDB，mongoDB，file这三个类必须实现这个接口所要求的功能，至于其他[锦上添花](https://www.baidu.com/s?wd=锦上添花&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)的功能，随你怎么发挥。而cache的代码，好像没有变，因为cache不需要关心数据是怎么存入数据库或者文件中。而cache的调用者，也不需要关心cache具体是怎么实现的，只要根据接口实现相应的方法就行了。多人协作你会更加受益，你们只需要商定好接口，然后分别实现就行了。

这就是依赖注入的魅力所在了，虽然看似如此简单。

以上代码还可以继续改进，直到你认为无可挑剔为止。比如，redis服务在一个请求中可能会调用多次，而每次调用都会重新创建，这将有损性能。只需扩展一下DI容器就好增加一个参数或增加一个方法，随你。

```php
class Di
{
    protected $_service = [];
    protected $_sharedService = [];
    public function set($name, $definition, $shared = false)
    {
        if ($shared) {
            $this->_sharedService[$name] = $definition;
        } else {
            $this->_service[$name] = $definition;
        }
    }
    public function get($name) {
        if (isset($this->_service[$name])) {
            $definition = $this->service[$name];
        } else if ($this->_sharedService[$name]) {
             $definition = $this->_sharedService[$name];
        } else {
            throw new Exception("Service '" . name . "' wasn't found in the dependency injection container");
        }
        ...
    }
```

这样以来，如果某个服务在一次请求中要调用多次，你就可以将shared属性设置为true，以减少不必要的浪费。如果你觉得每次在注入时都要setDi有点繁琐，想让他自动setDi，那可以这么做：

```php
interface DiAwareInterface
{
    public function setDI($di);
    public function getDI();
}

class Di
{
    protected $service;

    public function set($name, $definition)
    {
        $this->service[$name] = $definition;
    }

    public function get($name)
    {
        ...
        if (is_object($definition)) {
            $instance = call_user_func($definition);
        }

        // 如果实现了DiAwareInterface这个接口，自动注入
        if (is_object($instance)) {
            if ($instance instanceof DiAwareInterface) {
                $instance->setDI($this);
            }
        }

        return $instance;
    }
}

class redisDB implements BackendInterface, DiAwareInterface
{
    public function find($key, $lifetime) { }
    public function save($key, $value, $lifetime) { }
    public function delete($key) { }
}
```

然后，就可以这样：

```php
$di->set('cache', function() {
    return new cache([
        'connect' => 'mongo'
    ]);
});
```

我们现在所实现的这个DI容器还很简陋，还不支持复杂的注入，你可以继续完善它。

不过，通过这些代码你已经了解什么是依赖在注入了，你可以将这种思想应用到你的项目中，或者着手开发你自己的框架。