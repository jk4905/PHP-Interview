### Laravel 的生命周期 
#### PHP 的生命周期
PHP 有两种运行模式，web 模式和 cli（命令行） 模式。
1. 当我们在终端中敲 PHP 命令时使用的是命令行模式。
2. 而当使用 nginx 或者别的服务器作为宿主时，使用的事 web 模式。

当我们请求一个 php 文件时，比如 laravel 中的 public\index.php 文件，php 为了完成这个请求，经历了 5 个步骤：
1. 模块初始化（MINIT），即调用 php.ini 中指明的扩展的初始化函数的初始化工作，如 mysql 扩展。
2. 请求初始化（RINIT），即初始化执行本次脚本的所需要的变量名和变量值内容的符号表，如 $_SESSION 变量
3. 执行该 PHP 脚本
4. 请求处理完成（Request Shutdown）,按顺序调用各个模块的 RSHUTDOWN 方法，对每个变量调用 unset 函数，如 unset $_SESSION 变量。
5. 关闭模块（Module Shutdown），PHP 调用每个扩展的 MSHUTDOWN 方法，这是各个模块最后一次释放内存的机会，这意味着没有下一个请求。

web 模式与 CLI 模式的不同：
1. CLI 模式每次都会执行完整的 5 个周期，因为脚本不执行完不会有下一个请求。
2. WEB 为了应对并发，可能采用多线程，因此生命周期 1 5 只执行了一次，2-4 阶段重复执行，节省系统模块初始化所带来的开销。

![](http://pzjwh5v7g.bkt.clouddn.com/mweb/15719180600121.jpg)


作用：
优化 laravel 的代码，并且更加了解 singleton（单例）模式。每次请求，php 最后都会将变量 unset，所以 singleton 只会存在一个请求中，请求之间不能共享。因此，记住 PHP 是一个脚本语言，所有变量只会在一次请求中生效，下次请求时会被充值，而不像 java 静态变量拥有全局作用。


#### Laravel 的生命周期
Laravel 的请求周期是用处 public\index.php 开始，从 public\index.php 结束。
![](http://pzjwh5v7g.bkt.clouddn.com/mweb/15719192533214.jpg)

public\index.php 总共执行了四个步骤：
```php
// 1. 执行 composer 的自动加载，自动生成 class loader，包括 composer require 的所有依赖
require __DIR__.'/../bootstrap/autoload.php';

// 2. 生成容器 Container，Applition 实例，并向容器中注册核心组件（HttpKernel，ConsoleKernel, ExceptionHandle）
$app = require_once __DIR__.'/../bootstrap/app.php';
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

// 3. 处理请求，生成并发送响应。
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);
$response->send();

// 4. 请求结束，执行回调。（可终止中间件，就在这里回调）
$kernel->terminate($request, $response);
```

![](http://pzjwh5v7g.bkt.clouddn.com/mweb/15719197898521.jpg)


#### 启动 Laravel 的基础服务
1. 第一步注册加载 composer 自动生成 class loader 就是加载初始化第三方依赖。不属于 Laravel 内核。（spl_autoload_register 注册给定函数，作为 __autoload() 的实现）
2. 第二步生成容器 container，并向容器内注入核心组件，因为牵涉到容器 container 和合同 Contracts 后面详讲。
3. 第三步（重点），处理请求，生成并发送响应。
    1. 首先 laravel 框架会捕获到用户发送到 public\index.php 的请求。生成 Illuminate\Http\Request 实例，传递给 handle 方法。
    2. 在方法内部，将该 $request 实例绑定到第二步生成的容器中。
    3. 然后在该请求真正处理之前，调用 bootstrap 方法，进行必要的加载和注册，如环境监测，加载配置，注册 Facades (门面)，注册服务提供者，启动服务提供者等等。这是一个启动数组，具体在 Illuminate\Foundation\Http\Kernel 中，包括：
       ```php
       
       protected $bootstrappers = [
    'Illuminate\Foundation\Bootstrap\DetectEnvironment',
    'Illuminate\Foundation\Bootstrap\LoadConfiguration',
        'Illuminate\Foundation\Bootstrap\ConfigureLogging',
        'Illuminate\Foundation\Bootstrap\HandleExceptions',
        'Illuminate\Foundation\Bootstrap\RegisterFacades',
        'Illuminate\Foundation\Bootstrap\RegisterProviders',
        'Illuminate\Foundation\Bootstrap\BootProviders',
    ];
       ``` 
    laravel 是按顺序遍历执行注册这些基础服务的，注意顺序：facades 先于 ServiceProviders ，注册 facades 就是注册了 config\app.php 中的 aliases 数组，你使用的很多类，如 Auth，Cache, DB 等等都是Facades；而 ServiceProviders 的 register 方法永远在 boot 之前，以避免 boot 方法依赖某个实例而实例未注册的现象。

![Laravel 请求周期](http://pzjwh5v7g.bkt.clouddn.com/mweb/Laravel 请求周期.png)


#### 将请求传递给路由
到目前为止，laravel 还没有执行到你写的主要代码（除 ServiceProviders 中以外），因为还没有将请求传递给路由

在 laravel 基础服务启动后，就要把请求传给路由。传递路由是通过 pipeline 来传递，在 pipeline 有一堵墙，在传递路由之前的请求都要经过，这个墙被定义在 app\Http\Kernel.php 中的 $middleware 数组中，这就是中间件。默认只有一个 CheckForMaintenanceMode 中间件，用于检测你的网站是否暂时关闭。这是一个全局中间件，所有的请求都要经过，也可以自定义全局中间件。

然后开始遍历所有注册的路由，找到第一个符合条件的路由，经过它的路由中间件，进入到控制器或者闭包函数，执行你的代码。

所以所有的请求到达你写的代码之前，都经过了重重检测，确保不符合和恶意的请求被 laravel 拒之门外。
![](http://pzjwh5v7g.bkt.clouddn.com/mweb/15719236716200.jpg)

#### 服务容器
服务容器就是个普通的容器，用来装类的实例，然后需要用时在取出来。服务容器实现了依赖反转（Inversion of Control，缩写为IoC）。

正常情况下 A 需要用 B 要手动 new 个 B，意味着需要知道 B 的细节。比如构造函数等，但是随着项目变大，这种依赖是毁灭性的。依赖反转的意思是，将 A 主动获取 B 类的过程颠倒过来，类 A 只需要声明它需要什么，然后由容器提供。

![](http://pzjwh5v7g.bkt.clouddn.com/mweb/15719241677696.jpg)

这样做的好处是，A 不再依赖 B 的实现，一定程度上解决了耦合。

在 Laravel 中实现依赖反转有两种：
1. 依赖注入
2. 绑定

##### 依赖注入
```php
class UserController extends Controller
{
    /**
     * The user repository implementation.
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * Create a new controller instance.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
}
```

这里在 UserController 中需要一个 UserRepository 的实例，我们只需要在构造方法中声明我们需要的类型，容器在实例化 UserController 时会自动生成 UserRepository 实例。这样也就避免了了解 UserRepository 的细节和产生的依赖。

##### 绑定
绑定操作一般在 ServiceProviders 中的 register 方法中，最基本的是 bind 方法，它接受一个类名和闭包函数来获取实例：
```php
$this->app->bind('XblogConfig', function ($app) {
    return new MapRepository();
});
```
还有一个 singleton 方法，单例模式下绑定。

当然，也可以绑定一个已存在的对象到实例中：
```php
$this->app->instance('request',$request);
```
绑定之后，可以通过下面几种方式获取：
```php
app('requrest');

app()['request'];

app()->make('request');

resolve('request');
```

bind 方法和 singleton 方法唯一的区别是，bind 方法的闭包都会每次调用以上 4 种方法之一时执行，而 singleton 方法只会执行一次。
如果想每一个类都获取不同实例，或者需要“个性化”的实例时，这个时候需要 bind 方法，以免这次使用会对下一次使用造成干扰。
如果实例化一个比较费时或者不依赖生成的上下文，可以使用 singleton 方法绑定，其好处是如果在某一次请求中多次使用了某个类，那么只实例化一次会节省空间和时间。
```php
$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);
```

在 laravel 生命周期的第二部，laravel 默认（在bootstrap\app.php文件中）绑定了 Illuminate\Contracts\Http\Kernel，Illuminate\Contracts\Console\Kernel，Illuminate\Contracts\Debug\ExceptionHandler 接口的实现类。

还有一种上下文绑定，就是相同的接口，在不同类中可以自动获取不同的实现：
```php
$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });

$this->app->when(VideoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```
在上述声明中，同样的接口 Filesystem ，使用依赖注入时，PhotoController 获取的是 local 实例而 VideoController 获取的是 S3 实例。

#### Contracts & Facades（合同&门面）
laravel 的一个强大之处是，在配置文件中声明缓存类型(redis，memcached，file......) laravel 就会自动帮你切换成这种驱动了，而不需要修改逻辑和代码。laravel 定义了一系列的 Contracts ，本质是一些 php 接口，一系列标准，用来解耦具体需求对实现的依赖关系。
![](http://pzjwh5v7g.bkt.clouddn.com/mweb/15719267561310.jpg)
上图在不是用 Contracts 时，对于一种逻辑，只会有一种结果。如果需求变更，就需要重构代码和逻辑。
但是在使用 Contracts 之后，我们只需要按照接口写好逻辑，然后提供不同的实现，就可以再不动代码和逻辑的情况下得到更加多态的结果。

例子：
定义好接口：
```php
namespace App\Contracts;
use Closure;
interface XblogCache
{
    public function setTag($tag);
    public function setTime($time_in_minute);
    public function remember($key, Closure $entity, $tag = null);
    public function forget($key, $tag = null);
    public function clearCache($tag = null);
    public function clearAllCache();
}
```

实现具体缓存。
```php
class Cacheable implements XblogCache
{
public $tag;
public $cacheTime;
public function setTag($tag)
{
    $this->tag = $tag;
}
public function remember($key, Closure $entity, $tag = null)
{
    return cache()->tags($tag == null ? $this->tag : $tag)->remember($key, $this->cacheTime, $entity);
}
public function forget($key, $tag = null)
{
    cache()->tags($tag == null ? $this->tag : $tag)->forget($key);
}
public function clearCache($tag = null)
{
    cache()->tags($tag == null ? $this->tag : $tag)->flush();
}
public function clearAllCache()
{
    cache()->flush();
}
public function setTime($time_in_minute)
{
    $this->cacheTime = $time_in_minute;
}
}
```
不使用缓存：
```php
class NoCache implements XblogCache
{
public function setTag($tag)
{
    // Do Nothing
}
public function setTime($time_in_minute)
{
    // Do Nothing
}
public function remember($key, Closure $entity, $tag = null)
{
    /**
     * directly return
     */
    return $entity();
}
public function forget($key, $tag = null)
{
    // Do Nothing
}
public function clearCache($tag = null)
{
    // Do Nothing
}
public function clearAllCache()
{
    // Do Nothing
}
}
```
再利用容器绑定，根据不同配置获得不同结果：
```php
public function register()
{
        $this->app->bind('XblogCache', function ($app) {
            if (config('cache.enable') == 'true') {
                return new Cacheable();
            } else {
                return new NoCache();
            }
        });
}
```
实际上，Laravel所有的核心服务都是实现了某个 Contracts 接口（都在Illuminate\Contracts\文件夹下面），而不是依赖具体的实现，所以完全可以在不改动框架的前提下，使用自己的代码改变 Laravel 框架核心服务的实现方式。

Facades,在我们把类的实例绑定到容器的时候相当于给类起了个别名，然后覆盖 Facade 的静态方法 getFacadeAccessor 并返回你的别名，然后你就可以使用你自己的 Facade 的静态方法来调用你绑定类的动态方法了。其实 Facade 类利用了 __callStatic() 这个魔术方法来延迟调用容器中的对象的方法，这里不过多讲解，你只需要知道 Facade 实现了将对它调用的静态方法映射到绑定类的动态方法上，这样你就可以使用简单类名调用而不需要记住长长的类名。这也是 Facades 的中文翻译为假象的原因。



#### 总结
Laravel 提供了强大的脚手架，如 orm，carbon 时间处理。laravel 的核心是容器和抽象解耦，实现高扩展性。

学习 Laravel 的设计模式和思想：
1. 理解 laravel 的生命周期和请求生命周期的概念。
2. 所有的静态变量和单例，都会在下一次请求到来时重新初始化。
3. 将耗时且调用频繁的类用 singleton 绑定。
4. 将变化的选项抽象为 Contracts，依赖接口而不依赖具体实现。
5. 善于引用 laravel 提供的容器。

#### 参考
[Laravel的核心概念](https://lufficc.com/blog/the-core-conception-of-laravel)
