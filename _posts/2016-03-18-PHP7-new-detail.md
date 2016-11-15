---
layout: post
title: PHP7新特性总结
description: "描述PHP7新特性。PHP7新特性总结"
modified: 2016-03-18
tags: [php]
categories: [intro]
---
<figure>
	<a href="#"><img src="http://pic.chinaz.com/2016/0126/20160125165737_60612.jpg" alt=""></a>
</figure>

PHP7版本相较于老版本，加了大量新特性，同时性能得到了显著提升：是PHP5.6性能的2倍，在wordpress表
现上超过了HHVM。下面来总结下PHP7新增的特性。
# 一、新增特性

## 1、标量类型声明
现在可以使用下列类型参数（无论用强制模式还是严格模式）：字符串(string),整数 (int), 浮点数
 (float), 以及布尔值 (bool)。它们扩充了PHP5中引入的其他类型：类名，接口，数组和回调类型。
 其有两种模式：强制模式和严格模式。
*强制模式
{% highlight ruby %}
function get(int $bool){
var_dump($bool);
}
get(true);//将显示int(1)
{% endhighlight %}
*严格模式
在文件顶部添加：
{% highlight ruby %}
declare(strict_types=1);
{% endhighlight %}
在此模式下，若参数与所声明类型不符合，则会触发致命错误。

## 2、返回值类型声明
PHP 7 增加了对返回类型声明的支持。类似于参数类型声明，返回类型声明指明了函数返回值的类型。
可用的类型与参数声明中可用的类型相同。同时也包含强制模式和严格模式。
{% highlight ruby %}
function arraysSum(array ...$arrays): array{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}
print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
//将会输出
Array

(

[0] => 6

[1] => 15

[2] => 24

)
{% endhighlight %}

## 3、null 合并运算符
新增了 null 合并运算符 (??)。
{% highlight ruby %}
$username = $_GET['user'] ?? 'nobody';
{% endhighlight %}
等价于
{% highlight ruby %}
$username = isset($_GET['user']) ? $_GET['user'] : 'nobody';
{% endhighlight %}

## 4、太空船操作符（组合比较符）
太空船操作符（<=>）用于比较两个表达式。当操作符左边小于、等于或大于右边时它分别返回-1、0或1。
{% highlight ruby %}
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1
{% endhighlight %}

## 5、通过 define() 定义常量数组
{% highlight ruby %}
define('ANIMALS', [
    'dog',
    'cat',
    'bird'
]);
{% endhighlight %}

## 6、匿名类
现在支持通过new class 来实例化一个匿名类，这可以用来替代一些“用后即焚”的完整类定义。
{% highlight ruby %}
interface Logger {
    public function log(string $msg);
}

class Application {
    private $logger;

    public function getLogger(): Logger {
         return $this->logger;
    }

    public function setLogger(Logger $logger) {
         $this->logger = $logger;
    }
}

$app = new Application;
$app->setLogger(new class implements Logger {
    public function log(string $msg) {
        echo $msg;
    }
});

var_dump($app->getLogger());
{% endhighlight %}

## 7、Unicode codepoint 转译语法
可将16进制形式的 Unicode codepoint，转换并打印出一个双引号或heredoc包围的 UTF-8 
编码格式的字符串。可以接受任何有效的 codepoint，并且开头的 0 是可以省略的。
{% highlight ruby %}
echo "\u{aa}";    //ª
echo "\u{0000aa}";//ª
echo "\u{9999}";  //香
{% endhighlight %}

## 8、Closure::call()
Closure::call() 现在有着更好的性能，简短干练的暂时绑定一个方法到对象上闭包并调用它。
{% highlight ruby %}
class A {private $x = 1;}
$getX = function() {return $this->x;};
echo $getX->call(new A);//1
{% endhighlight %}

## 9、为 unserialize() 提供过滤
这个特性旨在提供更安全的方式解包不可靠的数据。它通过白名单的方式来防止潜在的代码注入。
{% highlight ruby %}
$data = unserialize($foo, ["allowed_classes" => ["MyClass", "MyClass2"]]);
{% endhighlight %}

## 10、IntlChar
用于暴露出更多的 ICU（是一套稳定成熟、功能强大、轻便易用和跨平台支持Unicode 的开发包） 功能。
{% highlight ruby %}
printf('%x', IntlChar::CODEPOINT_MAX);//10ffff
echo IntlChar::charName('@');         //COMMERCIAL AT
var_dump(IntlChar::ispunct('!'));     //bool(true)
{% endhighlight %}
需安装intl扩展

## 11、预期
预期是向后兼用并增强之前的 assert()（检查一个断言是否为false） 的方法。 
它使得在生产环境中启用断言为零成本，并且提供当断言失败时抛出特定异常的能力。
老版本的API出于兼容目的将继续被维护，assert()
现在是一个语言结构，它允许第一个参数是一个表达式，而不仅仅是一个待计算的 
string或一个待测试的boolean。

## 12、使用use声明组
从同一 namespace 导入的类、函数和常量现在可以通过单个 use 语句 一次性导入了。
{% highlight ruby %}
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
{% endhighlight %}

## 13、生成器的返回值
在 PHP7 中，当生成器迭代完成后，可以获取该生成器函数的返回值。通过 Generator::getReturn() 
得到。
{% highlight ruby %}
$gen = (function() {
    yield 1;
    yield 2;

    return 3;
})();

foreach ($gen as $val) {
    echo $val, PHP_EOL;
}

echo $gen->getReturn(), PHP_EOL;
//将会输出
1 2 3
{% endhighlight %}

## 14、迭代器中引用迭代器
在生成器中可以引入另一个或几个生成器。
{% highlight ruby %}
function gen()
{
    yield 1;
    yield 2;

    yield from gen2();
}

function gen2()
{
    yield 3;
    yield 4;
}
{% endhighlight %}

## 15、获取整数部分的新函数intdiv()
接收两个参数作为被除数和除数，返回他们相除结果的整数部分。
{% highlight ruby %}
var_dump(intdiv(10, 3));//int(3)
{% endhighlight %}

## 16、Session options
session_start() 函数可以接收一个数组作为参数，可以覆盖 php.ini 中 session 的配置项。
比如，把 cache_limiter 设置为私有的，同时在阅读完 session 后立即关闭。
{% highlight ruby %}
session_start([
'cache_limiter' => 'private',
'read_and_close' => true,
]);
{% endhighlight %}

## 17、preg_replace_callback_array() 
新增了一个函数 preg_replace_callback_array() ，使用该函数可以使得在使用 
preg_replace_callback() 函数时代码变得更加优雅。

## 18、CSPRNG
新增两个函数 : random_bytes() and random_int()。可使得随机数变得安全了。
random_bytes() — 加密生存被保护的伪随机字符串
random_int() — 加密生存被保护的伪随机整数

## 19、list()可支持数组式访问接口（ArrayAccess）

# 二、不兼容性
## 1、foreach 不再改变内部数组指针

## 2、foreach 通过引用遍历时，有更好的迭代特性
当使用引用遍历数组时，现在 foreach 
在迭代中能更好的跟踪变化。例如，在迭代中添加一个迭代值到数组中，参考下面的代码：
{% highlight ruby %}
$array = [0];
foreach ($array as &$val) {
var_dump($val);
$array[1] = 1;
}

PHP5 输出：
int(0)

PHP7 输出：
int(0)
int(1)
{% endhighlight %}

## 3、十六进制字符串不再被认为是数字

## 4、new 操作符创建的对象不能以引用方式赋值给变量

## 5、移除了 ASP 和 script PHP 标签
受到影响的标签有： `<% %> 、 <%= %> 、 <script language="php"> </script>`

## 6、在数值溢出的时候，内部函数将会失败
将浮点数转换为整数的时候，如果浮点数值太大，导致无法以整数表达的情况下， 
在之前的版本中，内部函数会直接将整数截断，并不会引发错误。 在 PHP 7.0 
中，如果发生这种情况，会引发 E_WARNING 错误，并且返回 NULL 。

## 7、JSON 扩展已经被 JSOND 取代
JSON 扩展已经被 JSOND 扩展取代。 对于数值的处理，有以下两点需要注意的： 
第一，数值不能以点号（ . ）结束 （例如，数值 34. 必须写作 34.0 或 34 ）。 
第二，如果使用科学计数法表示数值， e 前面必须不是点号（ . ） （例如， 3.e3 必须写作 3.0e3
 或 3e3 ）。

## 8、INI 文件中 # 注释格式被 ; 替换

## 9 、 yield 变更为右联接运算符
{% highlight ruby %}
echo yield -1;
// 在之前版本中会被解释为：
echo (yield) - 1;
// 现在，它将被解释为：
echo yield (-1);

yield $foo or die;
// 在之前版本中会被解释为：
yield ($foo or die);
// 现在，它将被解释为：
(yield $foo) or die;
{% endhighlight %}

## 10、$HTTP_RAW_POST_DATA 被移除，请使用 php://input 作为替代。

参考：


1、http://php.net/manual/en/migration70.new-features.php#migration70.new-features.return-type-declarations

2、http://www.tuicool.com/articles/yARJRjQ