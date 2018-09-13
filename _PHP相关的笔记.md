[info]: # ({"title":"PHP相关笔记", "create":"2018-08-31 17:04:55", "modify":"2018-08-31 18:51:22", "category":"笔记", "tag_list":["PHP", "编程", "脚本", "最好的语言???"], "info_list":[]})

PHP相关的笔记基本都在这里，~~PHP是世界上最好的语言？？？~~

[preview]: # (end preview)

## php语法

### 格式

PHP脚本以`<?php`开始，以`?>`结束  
每条语句后跟`;`结束

```php
<!DOCTYPE html>
<html>
<body>
<h1>html element</h1>

<?php
// 代码区
echo "Hello World!";
?>

// 单行注释

/*
这是
多行
注释
*/

</body>
</html>
```

### 变量

以`$`开头，后接变量名  
变量名以字母、下划线开头，[a-z,A-Z,0-9]，不含空格  
变量名区分大小写

```php
<?php
$txt="hello world";
$x=6;
$y=10.5;
?>
```

### 作用域

local global static parameter

local & global:

```php
<?php
$a=1;
$x=5; // 全局变量
function test(){
    global $x; // 要在一个函数中访问一个全局变量，需要使用global关键字
    $GLOBALS['a']=$GLOBALS['x']+$GLOBALS['a'];
    $y=10; // 局部变量
}
?>
```

PHP将所有全局变量存储在一个名为`$GLOBALS[index]`的数组中。`index`保存变量的名称。这个数组可以在函数内部访问，也可以直接用来更新全局变量

static:

一个函数完成时，它的所有变量通常都会被删除。当希望某个局部变量不要被删除时，使用static

```php
<?php
function test(){
    static $x=0; // 仍是局部变量
    echo $x;
    $x++;
}
test();
test();
test();
// 输出 0 1 2
?>
```

parameter:

```php
<?php
function test($x){ // 参数变量
    echo $x;
}
test(2); // 输出 2
?>
```

### 常用输出

`echo`可以输出多个字符，无返回值，速度较快  
`print`只能输出一个字符，返回值1，速度较慢  
`PHP_EOL`是一个换行符，兼容较多平台

```php
<?php
echo 'hello', ' ', 'world';
echo('hello' . ' ' . 'world');

print 'hello world';
print('hello world');
?>
```

### EOF使用

必须后接分号  
`EOF`可以用任意其它字符代替，只需保证结束标识与开始标识一致  
结束标识必须顶格占一行  
开始标识可以不带引号或带单双引号，不带引号与带双引号解释内嵌的变量和转义符号，带单引号则不解释内嵌的变量和转义符号  
当内容需要内嵌引号时，不需要加转义符，本身对单双引号转义

```php
<?php
echo <<<EOF
    first line.
    second line.
    quote'a',"d".
EOF;
// 结束需要独立一行且前后不能空格
?>

<?php
$tmp='hello world';
$str= <<<EOF
    first line$tmp
    second line$tmp
    quote'a',"d"$tmp
EOF;
// 结束需要独立一行且前后不能空格
// 变量可以被正常解析，但是函数不可以。变量不需要用连接符 . 或 , 来拼接
echo $str;
?>
```

### 数据类型

String,Integer,Float,Boolean,Array,Object,NULL

String:

一个字符串是一串字符的序列，可以将任何文本放在单引号和双引号中

```php
<?php
$str="string";
$str='string';
$str='str' . 'ing';
?>
```

Integer:

整数必须至少有一个数字[0-9]  
整数不能包含逗号或空格  
整数是没有小数点  
整数可以是正数或负数  
整型可以用三种格式来指定：十进制，十六进制`0x`为前缀，八进制`0`为前缀

```php
<?php
$x = 10;
var_dump($x);
$x = -10; // 负数
var_dump($x);
$x = 0xA; // 十六进制数
var_dump($x);
$x = 012; // 八进制数
var_dump($x);
$y = 1.5;
var_dump($y); // 单精度
?>
```

Float:

浮点数是带小数部分的数字，或是指数形式

```php
<?php
$y=2.4e3;
var_dump($y);
$y=8E-5;
var_dump($y);
$y = 1.5;
var_dump($y); // 单精度
?>
```

Bollean:

布尔型可以是TRUE或FALSE

```php
<?php
$z=true;
var_dump($z);
$z=True;
var_dump($z);
$z=TRUE;
var_dump($z);

$z=false;
var_dump($z);
$z=False;
var_dump($z);
$z=FALSE;
var_dump($z);
?>
```

Array:

数组可以在一个变量中存储多个值

```php
<?php
$a=array("1","2","33");
var_dump($a);
?>
```

Object:

对象必须声明  
使用class关键字声明类对象  
类是可以包含属性和方法的结构  
在类中定义数据类型，然后在实例化的类中使用数据类型

```php
<?php
class A{
  var $one;
  function __construct($one="green") {
    $this->one = $one;
  }
  function wtf() {
    return $this->one;
  }
}
$a=new A();
echo $a->wtf() . "\n";
?>
```

null:

为NULL类型，表示变量没有值

```php
 <?php
$z=null;
var_dump($z);
$z=Null;
var_dump($z);
$z=NULL;
var_dump($z);
?>
```

### 常量

常量是一个简单值的标识符，该值在脚本中不能改变  
常量由英文字母、下划线、和数字组成，但数字不能作为首字母，常量名不需要加`$`修饰符  
**常量在整个脚本中都可以使用**

```php
<?php
bool define ( string $name , mixed $value [, bool $case_insensitive = false ] )
// name:必选参数，常量名称，即标志符 value:必选参数，常量的值 case_insensitive:可选参数，TRUE则大小写不敏感，默认是大小写敏感
?>
```

```php
<?php
define("VARNAME","a string for test");
echo VARNAME; // "a string for test"
echo varname; // "varname"; warning提示以后版本会throw Error

define("VARNAME","a string for test 2",true);
echo VARNAME; // "a string for test 2"
echo varname; // "a string for test 2"
?>
```

魔术常量

```php
__LINE__ // 文件中的当前行号
__FILE__ // 文件的完整路径和文件名
__DIR__ // 文件所在的目录
__FUNCTION__ // 返回该函数被定义时的名字（区分大小写）
__CLASS__ // 返回该类被定义时的名字（区分大小写）
__TRAIT__ // 代码复用的一个方法
<?php
class Base {
    public function sayHello() {
        echo 'Hello ';
    }
}
trait SayWorld {
    public function sayHello() {
        parent::sayHello();
        echo 'World!';
    }
}
class MyHelloWorld extends Base {
    use SayWorld;
}
$o = new MyHelloWorld();
$o->sayHello(); // 输出"Hello World!"
?>
// -
<?php
trait Hello {
    public function sayHello() {
        echo 'Hello ';
    }
}
trait World {
    public function sayWorld() {
        echo 'World';
    }
}
class MyHelloWorld {
    use Hello, World;  
    public function sayExclamationMark() {
        echo '!';
    }
}
$o = new MyHelloWorld();
$o->sayHello();
$o->sayWorld();
$o->sayExclamationMark();
// 输出"Hello World!"
?>
// ---
__METHOD__ // 类的方法名，返回该方法被定义时的名字（区分大小写）
__NAMESPACE__ // 当前命名空间的名称（区分大小写），在编译时定义
```

### 字符串操作

`.`并置运算符，用于把两个字符串值连接起来

```php
<?php
$x="string 1";
$y="string 2";
$z=$x . $y;
echo $z; // "string 1string 2"
?>
```

`strlen()`函数，计算字符串长度，返回字符串长度

```php
<?php
echo strlen("hello world");
?>
```

`strpos()`函数，在字符串内查找一个字符或一段指定的文本，返回第一个匹配的字符位置 或 FALSE

```php
<?php
echo strpos("hello world","world");
?>
```

### 运算符

算术运算符

```php
+ // 加
- // 减
* // 乘
/ // 除
% // 取余
- // 取反
. // 并置
intdiv() // 整除
```

赋值运算符

```php
=
+=
-=
*=
/=
%=
.=
```

递增/递减运算符

```php
$x++
++$x
$y--
--$y
```

比较运算符

```php
==
=== // 绝对等于
!=
<> // 不等于
>
<
>=
<=
```

逻辑运算符

```php
and // 与
or // 或
xor // 异或
&& // 与
|| // 或
! // 非
```

数组运算符

```php
+ // 集合
==
=== // 恒等、顺序相同
!= // 不相等
<> // 不相等
!== // 不恒等
```

三元运算符

```php
(expr 1)?(expr 2[true]):(expr 3[false])

(expr1)?:(expr3) //在expr1为TRUE时返回expr1，否则返回expr3

(expr1)??(expr3) //在expr1不存在返回expr2，否则返回expr1
```

组合比较符

```php
// 整型
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// 浮点型
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1

// 字符串
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
```

### 分支代码块

if && if···elseif···else

```php
if(expr){
    exprs;
}elseif(expr){
    exprs;
}else{
    exprs;
}
```

switch // 允许贯穿

```php
switch($var){
    case "c1":
        exprs;
        break;
    case "c2":
        exprs;
        break;
    default:
        exprs;
}
```

### 数组

`count()`函数获取数组长度

数值数组

```php
$ary=array("n1","n2","n3");
// ---
$ary[0]="n1";
$ary[1]="n2";
$ary[2]="n3";
// ---
$arrlength=count($ary);
for($i=0;$i<$arrlength;++$i)
{
    echo $ary[$i] . PHP_EOL;
}
```

关联数组

```php
$ary=array("a"=>"35","b"=>"37","c"=>"43");
// ---
$ary['a']="35";
$ary['b']="37";
$ary['c']="43";
// ---
foreach($ary as $key=>$value){
    echo "key=" . $key . ", value=" . $value . PHP_EOL;
}
```

排序

```php
sort() // 对数组进行升序排列
rsort() // 对数组进行降序排列
asort() // 根据关联数组的值，对数组进行升序排列
ksort() // 根据关联数组的键，对数组进行升序排列
arsort() // 根据关联数组的值，对数组进行降序排列
krsort() // 根据关联数组的键，对数组进行降序排列
```

### 超级全局变量

```php
$GLOBALS
// 包含了全部变量的全局组合数组，变量的名字就是数组的键
$x = 75;
$y = 25;
$GLOBALS["z"] = $GLOBALS["x"] + $GLOBALS["y"];
echo $z;
// ---
$_SERVER
// 包含了头信息(header)、路径(path)、以及脚本位置(script locations)等等信息的数组
echo $_SERVER["PHP_SELF"];
echo $_SERVER["SERVER_NAME"];
echo $_SERVER["HTTP_HOST"];
echo $_SERVER["HTTP_REFERER"];
echo $_SERVER["HTTP_USER_AGENT"];
echo $_SERVER["SCRIPT_NAME"];
// ---
$_REQUEST
// 用于收集HTML表单提交的数据
<html>
<body>
<form method="post" action="<?php echo $_SERVER['PHP_SELF'];?>">
Name: <input type="text" name="fname">
<input type="submit">
</form>
<?php
$name = $_REQUEST['fname'];
echo $name;
?>
</body>
</html>
// ---
$_POST
// 收集表单数据
<html>
<body>
<form method="post" action="<?php echo $_SERVER['PHP_SELF'];?>">
Name: <input type="text" name="fname">
<input type="submit">
</form>
<?php
$name = $_REQUEST['fname'];
echo $name;
?>
</body>
</html>
$_GET
// 收集表单数据
<html>
<body>
<a href="test_get.php?param1=value1&param2=value2">Test $GET</a>
</body>
</html>
// test_get.php
<html>
<body>
<?php
echo "First " . $_GET["param1"] . " Second " . $_GET['param2'];
?>
</body>
</html>
// ---
$_FILES
$_ENV
$_COOKIE
$_SESSION
```

### 循环代码块

```php
while(expr){
    // do something;
}

do{
    // do something;
    // 至少执行一次
}while(expr);

for($i=0;$i<10;++$i){
    // do something;
    // 循环10次
}

foreach($array as $value){
    // do something;
}
foreach($array as $key=>$value){
    // do something;
}
```

### 函数

```php
function funcName($param){
    //do something;
    return $return_value;
}

funcName($var);
```

### 命名空间

默认情况下，所有常量、类和函数名都放在全局空间下  
命名空间通过关键字namespace 来声明  
如果一个文件中包含命名空间，它必须在其它所有代码之前声明命名空间

```php
<?php  
namespace ns1;
const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }

namespace ns2;
const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }

//建议使用大括号形式的语法
namespace ns1 {
    const CONNECT_OK = 1;
    class Connection { /* ... */ }
    function connect() { /* ... */  }
}

namespace ns2 {
    const CONNECT_OK = 1;
    class Connection { /* ... */ }
    function connect() { /* ... */  }
}
?>
```

将全局的非命名空间中的代码与命名空间中的代码组合在一起，只能使用大括号形式的语法  
全局代码必须用一个不带名称的`namespace`语句加上大括号括起来

```php
<?php
namespace ns1 {
    const CONNECT_OK = 1;
    class Connection { /* ... */ }
    function connect() { /* ... */  }
}

namespace { // 全局代码
    session_start();
    $a = ns1\connect();
    echo ns1\Connection::start();
}
?>
```

在声明命名空间之前唯一合法的代码是用于定义源文件编码方式的`declare`语句  
所有非PHP代码包括空白符都不能出现在命名空间的声明之前

```php
<?php
declare(encoding='UTF-8'); //定义多个命名空间和不包含在命名空间中的代码

namespace MyProject {
    const CONNECT_OK = 1;
    class Connection { /* ... */ }
    function connect() { /* ... */  }
}

namespace { // 全局代码
    session_start();
    $a = MyProject\connect();
    echo MyProject\Connection::start();
}
?>
// ---
<?php
namespace MyProject; // 命名空间前出现了“<html>”错误，命名空间必须是程序脚本的第一条语句
?>
```

子命名空间

```php
<?php
namespace ns1\sub\level; //声明分层次的单个命名空间
const CONNECT_OK = 1;
class Connection { /* ... */ }
function Connect() { /* ... */  }
?>
```

命名空间使用

```php
// 非限定名称，或不包含前缀的类名称。 $a=new foo(); 或 foo::staticmethod();
// 如当前命名空间是cns，foo将被解析为cns\foo；如果使用foo的代码是全局的，不包含在任何命名空间中的代码，则foo会被解析为foo
// 如果命名空间中的函数或常量未定义，则该非限定的函数名称或常量名称会被解析为全局函数名称或常量名称

// 限定名称，或包含前缀的名称。  $a=new subnamespace\foo(); 或 subnamespace\foo::staticmethod();
// 如果当前的命名空间是cns，foo会被解析为cns\subnamespace\foo；如果使用foo的代码是全局的，不包含在任何命名空间中的代码，foo会被解析为subnamespace\foo

// 完全限定名称，或包含了全局前缀操作符的名称。 $a=new \cns\foo(); 或 \cns\foo::staticmethod();
// foo总是被解析为代码中的文字名(literal name) currentnamespace\foo

// file1.php
<?php
namespace Foo\Bar\subnamespace;

const FOO = 1;
function foo() {}
class foo{
    static function staticmethod() {}
}
?>

// file2.php
<?php
namespace Foo\Bar;
include 'file1.php';

const FOO = 2;
function foo() {}
class foo{
    static function staticmethod() {}
}

/* 非限定名称 */
foo(); // 解析为函数 Foo\Bar\foo
foo::staticmethod(); // 解析为类 Foo\Bar\foo ，方法为 staticmethod
echo FOO; // 解析为常量 Foo\Bar\FOO

/* 限定名称 */
subnamespace\foo(); // 解析为函数 Foo\Bar\subnamespace\foo
subnamespace\foo::staticmethod(); // 解析为类 Foo\Bar\subnamespace\foo, 以及类的方法 staticmethod
echo subnamespace\FOO; // 解析为常量 Foo\Bar\subnamespace\FOO

/* 完全限定名称 */
\Foo\Bar\foo(); // 解析为函数 Foo\Bar\foo
\Foo\Bar\foo::staticmethod(); // 解析为类 Foo\Bar\foo, 以及类的方法 staticmethod
echo \Foo\Bar\FOO; // 解析为常量 Foo\Bar\FOO
?>
```

访问任意全局类、函数或常量，都可以使用完全限定名称

```php
<?php
namespace Foo;
function strlen() {}
const INI_ALL = 3;
class Exception {}
$a = \strlen("hello world"); // 调用全局函数strlen
$b = \INI_ALL; // 访问全局常量 INI_ALL
$c = new \Exception("error"); // 实例化全局类 Exception
?>
```

### 命名空间和动态语言特征

```php
// file1.php
<?php
class classname{
    function __construct(){
        echo __METHOD__,"\n";
    }
}
function funcname(){
    echo __FUNCTION__,"\n";
}
const constname = "global";
$a = 'classname';
$obj = new $a; // prints classname::__construct
$b = 'funcname';
$b(); // prints funcname
echo constant('constname'), "\n"; // prints global
?>
```

如果要将上面的代码转换到命名空间中，动态访问元素，必须使用完全限定名称（包括命名空间前缀的类名称）  
因为在动态的类名称、函数名称或常量名称中，限定名称和完全限定名称没有区别，因此其前导的反斜杠是不必要的

```php
// file2.php
<?php
namespace namespacename;
class classname{
    function __construct(){
        echo __METHOD__,"\n";
    }
}
function funcname(){
    echo __FUNCTION__,"\n";
}
const constname = "namespaced";
include 'file1.php';
$a = 'classname';
$obj = new $a; // 输出 classname::__construct
$b = 'funcname';
$b(); // 输出函数名
echo constant('constname'), "\n"; // 输出 global

/* 如果使用双引号，使用方法为 "\\namespacename\\classname"*/
$a = '\namespacename\classname';
$obj = new $a; // 输出 namespacename\classname::__construct
$a = 'namespacename\classname';
$obj = new $a; // 输出 namespacename\classname::__construct
$b = 'namespacename\funcname';
$b(); // 输出 namespacename\funcname
$b = '\namespacename\funcname';
$b(); // 输出 namespacename\funcname
echo constant('\namespacename\constname'), "\n"; // 输出 namespaced
echo constant('namespacename\constname'), "\n"; // 输出 namespaced
?>
```

### `namespace`和`__NAMESPACE__`

```php
<?php
namespace MyProject;
echo '"', __NAMESPACE__, '"'; // 输出 "MyProject"
?>

<?php
echo '"', __NAMESPACE__, '"'; // 输出 ""
?>

<?php
namespace MyProject;
function get($classname){
    $a = __NAMESPACE__ . '\\' . $classname;
    return new $a;
}
?>

// ---
<?php
namespace MyProject;
namespace\func(); // calls function MyProject\func()
namespace\sub\func(); // calls function MyProject\sub\func()
namespace\cname::method(); // calls static method "method" of class MyProject\cname
$a = new namespace\sub\cname(); // instantiates object of class MyProject\sub\cname
$b = namespace\CONSTANT; // assigns value of constant MyProject\CONSTANT to $b
?>
```

### 别名/导入

```php
<?php
namespace foo;
use My\Full\Classname as Another;
use My\Full\NSname; // 与 use My\Full\NSname as NSname 相同
// 导入一个全局类
use \ArrayObject;
$obj = new namespace\Another; // 实例化 foo\Another 对象
$obj = new Another; // 实例化 My\Full\Classname　对象
NSname\subns\func(); // 调用函数 My\Full\NSname\subns\func
$a = new ArrayObject(array(1)); // 实例化 ArrayObject 对象
// 如果不使用 "use \ArrayObject" ，则实例化一个 foo\ArrayObject 对象
?>

<?php
use My\Full\Classname as Another, My\Full\NSname;
$obj = new Another; // 实例化 My\Full\Classname 对象
NSname\subns\func(); // 调用函数 My\Full\NSname\subns\func
?>

<?php
use My\Full\Classname as Another, My\Full\NSname;
$obj = new Another; // 实例化一个 My\Full\Classname 对象
$a = 'Another';
$obj = new $a;      // 实际化一个 Another 对象
?>

<?php
use My\Full\Classname as Another, My\Full\NSname;
$obj = new Another; // 实例化 My\Full\Classname 类
$obj = new \Another; // 实例化 Another 类
$obj = new Another\thing; // 实例化 My\Full\Classname\thing 类
$obj = new \Another\thing; // 实例化 Another\thing 类
?>
```

### 后备全局函数/常量

在一个命名空间中，当 PHP 遇到一个非限定的类、函数或常量名称时，它使用不同的优先策略来解析该名称  
类名称总是解析到当前命名空间中的名称，因此在访问系统内部或不包含在命名空间中的类名称时，必须使用完全限定名称  
对于函数和常量来说，如果当前命名空间中不存在该函数或常量，PHP 会退而使用全局空间中的函数或常量

```php
<?php
namespace A\B\C;
class Exception extends \Exception {}
$a = new Exception('hi'); // $a 是类 A\B\C\Exception 的一个对象
$b = new \Exception('hi'); // $b 是类 Exception 的一个对象
$c = new ArrayObject; // 错误, 找不到 A\B\C\ArrayObject 类
?>

<?php
namespace A\B\C;
const E_ERROR = 45;
function strlen($str){
    return \strlen($str) - 1;
}
echo E_ERROR, "\n"; // 输出 "45"
echo INI_ALL, "\n"; // 输出 "7" - 使用全局常量 INI_ALL
echo strlen('hi'), "\n"; // 输出 "1"
if (is_array('hi')) { // 输出 "is not array"
    echo "is array\n";
} else {
    echo "is not array\n";
}
?>

<?php
namespace A\B\C;
/* 这个函数是 A\B\C\fopen */
function fopen() {
     /* ... */
     $f = \fopen(...); // 调用全局的fopen函数
     return $f;
}
?>
```

### 命名空间的顺序

1. 对完全限定名称的函数，类和常量的调用在编译时解析。例如 new \A\B 解析为类 A\B。
2. 所有的非限定名称和限定名称（非完全限定名称）根据当前的导入规则在编译时进行转换。例如，如果命名空间 A\B\C 被导入为 C，那么对 C\D\e() 的调用就会被转换为 A\B\C\D\e()。
3. 在命名空间内部，所有的没有根据导入规则转换的限定名称均会在其前面加上当前的命名空间名称。例如，在命名空间 A\B 内部调用 C\D\e()，则 C\D\e() 会被转换为 A\B\C\D\e() 。
4. 非限定类名根据当前的导入规则在编译时转换（用全名代替短的导入名称）。例如，如果命名空间 A\B\C 导入为C，则 new C() 被转换为 new A\B\C() 。
5. 在命名空间内部（例如A\B），对非限定名称的函数调用是在运行时解析的。例如对函数 foo() 的调用是这样解析的：
    1. 在当前命名空间中查找名为 A\B\foo() 的函数
    2. 尝试查找并调用 全局(global) 空间中的函数 foo()。
6. 在命名空间（例如A\B）内部对非限定名称或限定名称类（非完全限定名称）的调用是在运行时解析的。下面是调用 new C() 及 new D\E() 的解析过程：
    1. new C()的解析:
        1. 在当前命名空间中查找A\B\C类。
        2. 尝试自动装载类A\B\C。
    2. new D\E()的解析:
        1. 在类名称前面加上当前命名空间名称变成：A\B\D\E，然后查找该类。
        2. 尝试自动装载类 A\B\D\E。
7. 为了引用全局命名空间中的全局类，必须使用完全限定名称 new \C()。

```php
<?php
namespace A;
use B\D, C\E as F;

// 函数调用
foo();      // 首先尝试调用定义在命名空间"A"中的函数foo()
            // 再尝试调用全局函数 "foo"

\foo();     // 调用全局空间函数 "foo"

my\foo();   // 调用定义在命名空间"A\my"中函数 "foo"

F();        // 首先尝试调用定义在命名空间"A"中的函数 "F"
            // 再尝试调用全局函数 "F"

// 类引用
new B();    // 创建命名空间 "A" 中定义的类 "B" 的一个对象
            // 如果未找到，则尝试自动装载类 "A\B"

new D();    // 使用导入规则，创建命名空间 "B" 中定义的类 "D" 的一个对象
            // 如果未找到，则尝试自动装载类 "B\D"

new F();    // 使用导入规则，创建命名空间 "C" 中定义的类 "E" 的一个对象
            // 如果未找到，则尝试自动装载类 "C\E"

new \B();   // 创建定义在全局空间中的类 "B" 的一个对象
            // 如果未发现，则尝试自动装载类 "B"

new \D();   // 创建定义在全局空间中的类 "D" 的一个对象
            // 如果未发现，则尝试自动装载类 "D"

new \F();   // 创建定义在全局空间中的类 "F" 的一个对象
            // 如果未发现，则尝试自动装载类 "F"

// 调用另一个命名空间中的静态方法或命名空间函数
B\foo();    // 调用命名空间 "A\B" 中函数 "foo"

B::foo();   // 调用命名空间 "A" 中定义的类 "B" 的 "foo" 方法
            // 如果未找到类 "A\B" ，则尝试自动装载类 "A\B"

D::foo();   // 使用导入规则，调用命名空间 "B" 中定义的类 "D" 的 "foo" 方法
            // 如果类 "B\D" 未找到，则尝试自动装载类 "B\D"

\B\foo();   // 调用命名空间 "B" 中的函数 "foo"

\B::foo();  // 调用全局空间中的类 "B" 的 "foo" 方法
            // 如果类 "B" 未找到，则尝试自动装载类 "B"

// 当前命名空间中的静态方法或函数
A\B::foo();   // 调用命名空间 "A\A" 中定义的类 "B" 的 "foo" 方法
              // 如果类 "A\A\B" 未找到，则尝试自动装载类 "A\A\B"

\A\B::foo();  // 调用命名空间 "A" 中定义的类 "B" 的 "foo" 方法
              // 如果类 "A\B" 未找到，则尝试自动装载类 "A\B"
?>
```

### 面向对象

类定义：  
类使用 class 关键字后加上类名定义  
类名后的一对大括号({})内可以定义变量和方法  
类的变量使用 var 来声明, 变量也可以初始化值  
函数定义类似 PHP 函数的定义，但函数只能通过该类及其实例化的对象访问

```php
<?php
class Site {
  /* 成员变量 */
  var $url;
  var $title;
  
  /* 成员函数 */
  function setUrl($par){
     $this->url = $par;
  }
  
  function getUrl(){
     echo $this->url . PHP_EOL;
  }
  
  function setTitle($par){
     $this->title = $par;
  }
  
  function getTitle(){
     echo $this->title . PHP_EOL;
  }
}

$new_obj = new Site;
echo $new_obj->getTitle();
?>
```

构造函数`void __construct ([ mixed $args [, $... ]] )`

```php
function __construct( $par1, $par2 ) {
   $this->url = $par1;
   $this->title = $par2;
}
```

析构函数`void __destruct ( void )`

```php
<?php
class MyDestructableClass {
   function __construct() {
       print "构造函数\n";
       $this->name = "MyDestructableClass";
   }

   function __destruct() {
       print "销毁 " . $this->name . "\n";
   }
}
$obj = new MyDestructableClass();
?>
```

继承`extends`，不支持多继承

```php
class Child extends Parent {
   // 代码部分
}
<?php
// 子类扩展
class Child_Site extends Site {
   var $category;

    function setCate($par){
        $this->category = $par;
    }
  
    function getCate(){
        echo $this->category . PHP_EOL;
    }
}
```

重写/覆盖/override

```php
function parentFunc(){
    // do something;
}
```

访问控制

```php
public // （公有）：公有的类成员可以在任何地方被访问。
protected // （受保护）：受保护的类成员则可以被其自身以及其子类和父类访问。
private // （私有）：私有的类成员则只能被其定义所在的类访问。
// 类属性必须定义为公有，受保护，私有之一。如果用 var 定义，则被视为公有。

class MyClass{
    public $public = 'Public';
    protected $protected = 'Protected';
    private $private = 'Private';
    function printHello(){
        echo $this->public;
        echo $this->protected;
        echo $this->private;
    }
}
$obj = new MyClass();
echo $obj->public; // 这行能被正常执行
echo $obj->protected; // 这行会产生一个错误
echo $obj->private; // 这行也会产生一个错误
$obj->printHello(); // 输出 Public、Protected 和 Private

class MyClass2 extends MyClass{
    // 可以对 public 和 protected 进行重定义，但 private 而不能
    protected $protected = 'Protected2';
    function printHello(){
        echo $this->public;
        echo $this->protected;
        echo $this->private;
    }
}
$obj2 = new MyClass2();
echo $obj2->public; // 这行能被正常执行
echo $obj2->private; // 未定义 private
echo $obj2->protected; // 这行会产生一个错误
$obj2->printHello(); // 输出 Public、Protected2 和 Undefined
?>
```

方法的访问控制

```php
// 类中的方法可以被定义为公有，私有或受保护。如果没有设置这些关键字，则该方法默认为公有

class MyClass{
    // 声明一个公有的构造函数
    public function __construct() { }
    // 声明一个公有的方法
    public function MyPublic() { }
    // 声明一个受保护的方法
    protected function MyProtected() { }
    // 声明一个私有的方法
    private function MyPrivate() { }
    // 此方法为公有
    function Foo(){
        $this->MyPublic();
        $this->MyProtected();
        $this->MyPrivate();
    }
}
$myclass = new MyClass;
$myclass->MyPublic(); // 这行能被正常执行
$myclass->MyProtected(); // 这行会产生一个错误
$myclass->MyPrivate(); // 这行会产生一个错误
$myclass->Foo(); // 公有，受保护，私有都可以执行

class MyClass2 extends MyClass{
    // 此方法为公有
    function Foo2(){
        $this->MyPublic();
        $this->MyProtected();
        $this->MyPrivate(); // 这行会产生一个错误
    }
}
$myclass2 = new MyClass2;
$myclass2->MyPublic(); // 这行能被正常执行
$myclass2->Foo2(); // 公有的和受保护的都可执行，但私有的不行

class Bar{
    public function test() {
        $this->testPrivate();
        $this->testPublic();
    }
    public function testPublic() {
        echo "Bar::testPublic\n";
    }
    private function testPrivate() {
        echo "Bar::testPrivate\n";
    }
}

class Foo extends Bar{
    public function testPublic() {
        echo "Foo::testPublic\n";
    }
    private function testPrivate() {
        echo "Foo::testPrivate\n";
    }
}
$myFoo = new Foo();
$myFoo->test(); // Bar::testPrivate
                // Foo::testPublic
?>
```

接口

使用接口（interface），可以指定某个类必须实现哪些方法，但不需要定义这些方法的具体内容  
接口是通过 interface 关键字来定义的，就像定义一个标准的类一样，但其中定义所有的方法都是空的  
接口中定义的所有方法都必须是公有，这是接口的特性  
要实现一个接口，使用 implements 操作符，类中必须实现接口中定义的所有方法，否则会报错误，类可以实现多个接口，用逗号来分隔多个接口的名称

```php
<?php
// 声明一个'iTemplate'接口
interface iTemplate{
    public function setVariable($name, $var);
    public function getHtml($template);
}

// 实现接口
class Template implements iTemplate{
    private $vars = array();
    public function setVariable($name, $var){
        $this->vars[$name] = $var;
    }
    public function getHtml($template){
        foreach($this->vars as $name => $value) {
            $template = str_replace('{' . $name . '}', $value, $template);
        }
        return $template;
    }
}
```

常量

可以把在类中始终保持不变的值定义为常量。在定义和使用常量的时候不需要使用 $ 符号  
常量的值必须是一个定值，不能是变量，类属性，数学运算的结果或函数调用  
自 PHP 5.3.0 起，可以用一个变量来动态调用类。但该变量的值不能为关键字（如 self，parent 或 static）。

```php
<?php
class MyClass{
    const constant = '常量值';
    function showConstant() {
        echo  self::constant . PHP_EOL;
    }
}
echo MyClass::constant . PHP_EOL;
$classname = "MyClass";
echo $classname::constant . PHP_EOL; // 自 5.3.0 起
$class = new MyClass();
$class->showConstant();
echo $class::constant . PHP_EOL; // 自 PHP 5.3.0 起
?>
```

抽象类

任何一个类，如果它里面至少有一个方法是被声明为抽象的，那么这个类就必须被声明为抽象的  
定义为抽象的类不能被实例化  
被定义为抽象的方法只是声明了其调用方式（参数），不能定义其具体的功能实现  
继承一个抽象类的时候，子类必须定义父类中的所有抽象方法  
这些方法的访问控制必须和父类中一样（或者更为宽松），例如某个抽象方法被声明为受保护的，那么子类中实现的方法就应该声明为受保护的或者公有的，而不能定义为私有的
**子类方法可以包含父类抽象方法中不存在的可选参数**

```php
<?php
abstract class AbstractClass{
 // 强制要求子类定义这些方法
    abstract protected function getValue();
    abstract protected function prefixValue($prefix);
    // 普通方法（非抽象方法）
    public function printOut() {
        print $this->getValue() . PHP_EOL;
    }
}

class ConcreteClass1 extends AbstractClass{
    protected function getValue() {
        return "ConcreteClass1";
    }
    public function prefixValue($prefix) {
        return "{$prefix}ConcreteClass1";
    }
}

class ConcreteClass2 extends AbstractClass{
    public function getValue() {
        return "ConcreteClass2";
    }
    public function prefixValue($prefix) {
        return "{$prefix}ConcreteClass2";
    }
}
$class1 = new ConcreteClass1;
$class1->printOut();
echo $class1->prefixValue('FOO_') . PHP_EOL;
$class2 = new ConcreteClass2;
$class2->printOut();
echo $class2->prefixValue('FOO_') . PHP_EOL;
?>
// ---
<?php
abstract class AbstractClass{
    // 我们的抽象方法仅需要定义需要的参数
    abstract protected function prefixName($name);
}

class ConcreteClass extends AbstractClass{
    // 我们的子类可以定义父类签名中不存在的可选参数
    public function prefixName($name, $separator = ".") {
        if ($name == "Pacman") {
            $prefix = "Mr";
        } elseif ($name == "Pacwoman") {
            $prefix = "Mrs";
        } else {
            $prefix = "";
        }
        return "{$prefix}{$separator} {$name}";
    }
}
$class = new ConcreteClass;
echo $class->prefixName("Pacman"), "\n";
echo $class->prefixName("Pacwoman"), "\n";
?>
```

Static 关键字

声明类属性或方法为 static(静态)，就可以不实例化类而直接访问  
静态属性不能通过一个类已实例化的对象来访问（但静态方法可以）  
由于静态方法不需要通过对象即可调用，所以伪变量 $this 在静态方法中不可用  
静态属性不可以由对象通过 -> 操作符来访问  
自 PHP 5.3.0 起，可以用一个变量来动态调用类。但该变量的值不能为关键字 self，parent 或 static

```php
<?php
class Foo{
  public static $my_static = 'foo';
  public function staticValue() {
     return self::$my_static;
  }
}
print Foo::$my_static . PHP_EOL;
$foo = new Foo();
print $foo->staticValue() . PHP_EOL;
?>
```

Final 关键字

如果父类中的方法被声明为 final，则子类无法覆盖该方法。如果一个类被声明为 final，则不能被继承

```php
<?php
class BaseClass {
   public function test() {
       echo "BaseClass::test() called" . PHP_EOL;
   }
   final public function moreTesting() {
       echo "BaseClass::moreTesting() called"  . PHP_EOL;
   }
}

class ChildClass extends BaseClass {
   public function moreTesting() {
       echo "ChildClass::moreTesting() called"  . PHP_EOL;
   }
}
// 报错信息 Fatal error: Cannot override final method BaseClass::moreTesting()
?>
```

调用父类构造方法`parent::__construct()`，PHP不会在**子类的构造方法**中自动的调用父类的构造方法

```php
<?php
class BaseClass {
   function __construct() {
       print "BaseClass 类中构造方法" . PHP_EOL;
   }
}
class SubClass extends BaseClass {
   function __construct() {
       parent::__construct();  // 子类构造方法不能自动调用父类的构造方法
       print "SubClass 类中构造方法" . PHP_EOL;
   }
}
class OtherSubClass extends BaseClass {
    // 继承 BaseClass 的构造方法
}
// 调用 BaseClass 构造方法
$obj = new BaseClass();
// 调用 BaseClass、SubClass 构造方法
$obj = new SubClass();
// 调用 BaseClass 构造方法
$obj = new OtherSubClass();
/*
BaseClass 类中构造方法
BaseClass 类中构造方法
SubClass 类中构造方法
BaseClass 类中构造方法
*/
?>
```

## todo