### PHP 面试题

#### foreach 的用法，与引用 
```php
<?php
	$a = [1,2,3];
	foreach ($a as $key => &$value) {
		# code...
	}
	var_dump($a);

	foreach ($a as $key => $value) {
		var_dump($a);
		exit;
	}
	var_dump($a);
```
输出两次的值：
```
1 2 3
1 2 2
```

为什么？
因为在第一个循环中，\$value 加了引用符号，那么 \$value 引用值。
在第一个 foreach 中：
1. 第一次循环 \$value 是 \$a[0] 的引用。
2. 第二次循环 \$value 是 \$a[1] 的引用。
3. 第三次循环 \$value 是 \$a[2] 的引用。
而循环结束时 \$value 并没有被销毁，是 \$a[2] 的引用，所以对 \$value 赋值就是对 \$a[2] 赋值。

在第二个 foreach 中，继续为 \$value 赋值：
第一次循环，结束后 \$value = \$a[0] = \$a[2] = 1,所以此时的 \$a = [1,2,1];
第二次循环，结束后 \$value = \$a[1] = \$a[2] = 2,所以此时的 \$a = [1,2,2];
第二次循环，结束后 \$value = \$a[2] = \$a[2] = 2,所以此时的 \$a = [1,2,2];

参考：
[php 中的引用(&)与foreach结合后的一个注意点](https://blog.csdn.net/cn_yefeng/article/details/80168414)

#### 延迟动态绑定
```php
<?php
class A {
    public static function who() {
        echo __CLASS__;
    }
    public static function test() {
        static::who(); // 后期静态绑定从这里开始
    }
}

class B extends A {
    public static function who() {
        echo __CLASS__;
    }
}

B::test();
```
输出：B
原因：static:: 不再被解析为定义当前方法所在的类，而是运行时所在的类。

而 self:: 或 __CLASS__ 对当前类的引用，取决于定义当前方法的类。


```php
<?php
class A {
    public static function foo() {
        static::who();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}

class B extends A {
    public static function test() {
        A::foo();
        parent::foo();
        self::foo();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}
class C extends B {
    public static function who() {
        echo __CLASS__."\n";
    }
}

C::test();
```
输出 ACC


#### array+array 与 array_merge 区别
```php
<?php
$arr1 = ['a'=>'PHP', 'apache'];
$arr2 = ['a'=>'PHP2', 'MySQl', 'HTML', 'CSS'];
$mergeArr = array_merge($arr1, $arr2);
$plusArr = $arr1 + $arr2;
var_dump($mergeArr);
var_dump($plusArr);

echo PHP_EOL;
echo PHP_EOL;
echo PHP_EOL;
$arr1 = ['PHP', 'apache'];
$arr2 = ['PHP2', 'MySQl', 'HTML', 'CSS'];
$mergeArr = array_merge($arr1, $arr2);
$plusArr = $arr1 + $arr2;
var_dump($mergeArr);
var_dump($plusArr);
```
结果：
```php
array(5) {
  ["a"]=>
  string(4) "PHP2"
  [0]=>
  string(6) "apache"
  [1]=>
  string(5) "MySQl"
  [2]=>
  string(4) "HTML"
  [3]=>
  string(3) "CSS"
}
array(4) {
  ["a"]=>
  string(3) "PHP"
  [0]=>
  string(6) "apache"
  [1]=>
  string(4) "HTML"
  [2]=>
  string(3) "CSS"
}



array(6) {
  [0]=>
  string(3) "PHP"
  [1]=>
  string(6) "apache"
  [2]=>
  string(4) "PHP2"
  [3]=>
  string(5) "MySQl"
  [4]=>
  string(4) "HTML"
  [5]=>
  string(3) "CSS"
}
array(4) {
  [0]=>
  string(3) "PHP"
  [1]=>
  string(6) "apache"
  [2]=>
  string(4) "HTML"
  [3]=>
  string(3) "CSS"
}
[Finished in 0.0s]
```


区别：
1. array_merge 会重新给下标排序。array+array 不会
2. 当下标是数字时，array_merge 不会覆盖相同下标的值，而是按顺序重新排下标；array+array 则会以最先出现的值为结果，后面的值抛弃。
3. 当下标为字母时，array_merge 后面的会覆盖相同值。而 array+array 则跟之前一样，会以最开始出现的值为结果，后面的值抛弃。

#### unset 
```php
<?php
 $a = 1;
 $b = &$a;

 unset($b);
 var_dump($a);
```
输出结果：
```php
1
```
unset 只会销毁 \$b 的类型，没有修改 \$a。


##### 参考
[系统的讲解 - SSO单点登录](https://segmentfault.com/a/1190000019142622)

#### array_map 与 array_walk 区别
1. array_map 是 array_map('函数名','数组') ,而 array_walk 是 array_walk('数组'，'函数名')
2. array_map 可以使用自定义函数也可以使用系统函数，array_walk 只能使用系统函数
3. array_map 不改变原数组，而是得到一个新数组，而 array_walk 是改变原数组的值。
4. array_map 有返回值，array_walk 没有返回值。

foreach 的效率比 for 高。
array_walk 效率比 foreach 高。
