<!--
{
    "title": "c相关",
    "create": "2018-07-20 11:04:52",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "c"
    ],
    "info": []
}
-->

## 程序结构

```c
#include <stdio.h> /* 预编译指令 */

int main(int argc, char **argv){ /* 主函数，程序入口 */
    /* 这里是注释 */
    printf("Hello World \n"); /* 语句 */
    return 0; /* 终止&返回值 */
}
```

编写-编译-链接-执行

## 关键字

```c
auto
break
case
char
const
continue
default
do
double
else
enum
extern
float
for
goto
if
int
long
register
return
short
signed
sizeof /* 关键字/运算符/非函数 */
static
struct
switch
typedef
unsigned
union
void
volatile

/* C99 */
_Bool
_Complex
_Imaginary
inline
restrict

/* C11 */
_Alignas
_Alignof
_Atomic
_Generic
_Noreturn
_Static_assert
_Thread_local
```

## 数据类型

### 类型信息

- 基本类型
    - 整数类型
    - 浮点类型
- 枚举类型
- void类型
- 派生类型
    - 指针类型
    - 数组类型
    - 结构类型
    - 共用体类型
    - 函数类型

### 存储大小

```size
// win32&64/i686/x86_64
char 1/1/1 字节
short 2/2/2 字节
int 4/4/4 字节
long 4/4/8 字节
float 4/4/4 字节
double 8/8/8 字节
// 准确大小使用 sizeof(type)
```

### 书写格式

```c
10 20 /* 默认10进制 */
023 012 /* 以0开头为8进制 */
0b1101110 /* 以0b开头为2进制 */
0x234def /* 以0x开头为16进制 */
--------
float x = 2.3f /* 表示x为单精度浮点数 */
double y = 2.3 /* 表示双精度（默认浮点数常量为double类型） */
--------
'a', 'b', '\n', '\t' /* 字符型常量 */
--------
char *str="abcd"
```

### 数值计算

```c
#include <stdio.h>
int main(int argc, char **argv) {
    float f, x = 3.6f, y = 5.2f; /* 带f表示单精度浮点数 */
    int i = 4, a, b; /* 整型 */
    a = x + y; /* x+y为8.8 但a为int 故舍去小数部分 a为8 */
    b = (int) (x + y); /* 强制转换为int a为8 */
    f = 10 / i; /* i为int 故而10/i转为2 f值为2f */
    printf("a=%d,b=%d,f=%f,x=%f\n", a, b, f, x); /* a=8,b=8,f=2.000000,x=3.600000 */
    char *str="abcd"; /* str为指针  */
    printf("%s",str);
    return 0;
}
```

### 指针类型

```c
#include <stdio.h>
int main(int argc, char **argv) {
    char *str = "abc123";
    printf("---1.0---\n");
    printf("%%c *str => %c\n", *str);       /* %c *str => a */
    printf("---1.1---\n");
    printf("%%s str => %s\n", str);         /* %s str => abc123 */
    printf("%%d str => %d\n", str);         /* %d str => 4210736 */
    printf("%%x str => %x\n", str);         /* %x str => 404030 */
    printf("%%p str => %p\n", str);         /* %p str => 0000000000404030 */
    printf("---1.2---\n");
    printf("%%d &(*str) => %d\n", &(*str)); /* %d &(*str) => 4210736 */
    printf("%%x &(*str) => %x\n", &(*str)); /* %x &(*str) => 404030 */
    printf("---1.3---\n");
    printf("%%x &str => %x\n", &str);       /* %x &str => 62fe48 */
    printf("------------------\n");
    str += 2;
    printf("str+=2\n");
    printf("------------------\n");
    printf("---2.0---\n");
    printf("%%c *str => %c\n", *str);       /* %c *str => c */
    printf("---2.1---\n");
    printf("%%s str => %s\n", str);         /* %s str => c123 */
    printf("%%d str => %d\n", str);         /* %d str => 4210738 */
    printf("%%x str => %x\n", str);         /* %x str => 404032 */
    printf("%%p str => %p\n", str);         /* %p str => 0000000000404032 */
    printf("---2.2---\n");
    printf("%%d &(*str) => %d\n", &(*str)); /* %d &(*str) => 4210738 */
    printf("%%x &(*str) => %x\n", &(*str)); /* %x &(*str) => 404032 */
    printf("---2.3---\n");
    printf("%%x &str => %x\n", &str);       /* %x &str => 62fe48 */
    return 0;
}
```

## 进制转换

### 十进制转二进制

除2取余，逆序排列

```calc
19(10)
19/2 = 9···1
9/2 = 4···1
4/2 = 2···0
2/2 = 1···0
1/2 = 0···1

故而 19(10) = (10011)2
```

### 十进小数转二进

乘2取整，顺序排列

```calc
0.125(10)
0.125*2 = 0.25···0 & 0.25
0.25*2 = 0.5···0 & 0.5
0.5*2 = 1···1 & 0

故而 0.125(10) = (0.001)2
```

### 二进制转十进制

```calc
(110.11)2 = 1*2^2 + 1*2^1 + 0*2^0 + 1*2^-1 + 1*2^-2 = (6.75)10
```

## 内存中存储格式

- float格式：1位符号，8位指数，23位小数
    - 将实数的绝对值化为二进制
    - 将二进制格式实数的小数点左移或右移n位，直到小数点移动到第一个有效数字的右边
    - 从小数点右边第一位开始数出二十三位数字放入第22到第0位
    - 如果实数是正的，则在第31位放入“0”，否则放入“1”
    - 如果n是左移得到的，说明指数是正的，第30位放入“1”。如果n是右移得到的或n=0，则第30位放入“0”
    - 如果n是左移得到的，则将n减去1后化为二进制，并在左边加“0”补足七位，放入第29到第23位。如果n是右移得到的或n=0，则将n化为二进制后在左边加“0”补足七位，再各位求反，再放入第29到第23位
    - （即指数位为127+n）
- double格式：1位符号，11位指数，52位小数
    - 指数位1023+n
    - 其他类似float

```float
// 内存中32bit
      0      0000 0000      000 0000 0000 0000 0000 0000
// 正负1bit  指数位数8bit     小数位数23bit
// 即内存中 (0 0111 1111 100 0000 0000 0000 0000 0000) = (1.1)2 = (1.5)10
```

## c语言语法

### 中文输出问题

（源代码文件UTF-8，输出控制台GBK）

```c
#include <stdio.h> /* 源代码默认UTF-8 */
#include <stdlib.h> /* system() 函数头文件 */
int main() {
    system("chcp 65001"); /* windows下的控制台编码UTF-8/65001，默认GBK/936 */
    int var1 = 1;
    char *str1 = "中文";
    printf("变量var1的值 %d，内存地址 %p\n", var1, &var1); /* 不修改控制台编码则此处乱码 */
    printf("变量str1的值 %s，内存地址 %p\n", str1, &str1);
    return 0;
}
/* 输出：变量var1的值 1，内存地址 000000000062FE4C    变量str1的值 中文，内存地址 000000000062FE40 */
------
#include <stdio.h>
#include <locale.h>
#include <wchar.h>
int main(void) {
    setlocale(LC_CTYPE, ""); /* 留空，从当前系统获得默认的环境编码 或 setlocale(LC_CTYPE, "chs") */
    wchar_t wstr[] = L"中文"; /* 宽字符，会做转换进入二进制文件 */
    wprintf(L"其他 %ls",wstr);
    return 0;
}
/* wchar_t：文件编码 -> Unicode(win:UTF-16/lin:UTF-32) -> wprintf转成本地编码输出 */
/* 输出：其他 中文 */
```

### 内存区域

栈(存放局部变量，由操作系统自动释放) -> 堆(动态分配给程序的内存区域，由程序员手动释放) -> 数据区(常量区:存放常量,一般是字符串常量; 全局区/静态区:存放全局变量和静态变量) -> 代码区(存放可执行代码的区域) [高地址 -> 低地址]

```memory
high address
    命令行参数/环境变量
    栈↓
      ↓

      ↑
    堆↑
    数据区
        未初始化数据
        初始化数据
    代码区
low address
栈的生长方向和数组内元素的存放方向相反/堆的生长方向和数组内元素的存放方向相同
```

### 变量

```c
size_t size_t_var; /* 一种数据类型，容量范围一般大于int和unsigned，不涉及负值范围的表示size取值的，如array[size_t]等 */
extern int d; /* 声明变量，未建立存储空间 */
extern int a=1; /* 声明定义初始化，已建立存储空间 */
int i,j,k; /* 声明并定义变量，已建立存储空间 */
int c=1; /* 声明定义初始化，已建立存储空间 */
```

### 数值

```c
0x222 /* 十六进制数 */
0222 /* 八进制数 */
222 /* 默认十进制 */
222u /* u/U 无符号整数 */
222L /* l/L 长整数 */
```

### 定义

```c
#define <标识符> <值>
const <type> <identified> = <value>
```

### 存储类

```c
auto int a; /* auto是所有局部变量默认的存储类，只用于函数内，即只修饰局部变量 */
register int c; /* register存储于寄存器（取决于实际），不能用&运算符（无内存位置） */
static int g; /* static全局变量默认存储类，修饰局部变量可以在函数调用之间保持局部变量的值，修饰全局变量时，会使变量的作用域限制在声明它的文件内 */

void func(void){
    static int var_name=1; /* 每次调用函数时 var_name 值不会被重置 */
    ++var_name;
}

extern var_2; /* extern全局变量的引用，对所有的程序文件可见，通常用于当有两个或多个文件共享相同的全局变量或函数的时候 */
------
file1.c:
#include <stdio.h>
extern int count;
void some_extern_func(void)
{
   printf("count is %d\n", count);
}
---
file2.c:
#include <stdio.h>
int count;
extern void some_extern_func();
int main()
{
   count = 5;
   some_extern_func();
}
/* 编译: gcc file1.c file2.c => a.out => ./a.out ==> count is 5 */
```

### 函数

```c
static int max(int x, int b); /* 函数只能被本文件中其他函数所调用，称为内部函数/静态函数。函数的作用域只局限于所在文件，定义内部函数加static */
extern int max(int a, int b); /* 加关键字 xtern，则是外部函数，可供其它文件调用。默认为外部函数 */
```

### 变量作用域

- 局部变量：在某个函数或块的内部声明的变量称为局部变量，它们只能被该函数或该代码块内部的语句使用，局部变量在函数外部是不可见的
- 全局变量：定义在函数外部，全局变量在整个程序生命周期内都是有效的，在任意的函数内部能访问全局变量
- 形式参数：函数的参数，形式参数，被当作该函数内的局部变量，它们会优先覆盖全局变量
- 作用域：
    - 局部变量和全局变量的名称可以相同，但是在函数内，局部变量的值会覆盖全局变量的值
    - 形式参数，被当作该函数内的局部变量，它们会优先覆盖全局变量

```c
#include <stdio.h>
int a=10; /* 全局变量 */
int main(){
    int a=1; /* 局部变量 */
    int b=2;
    int c=0;
    int sum(int x, int y); /* x y为形参 */
    printf("%d\n",a);
    c=sum(a,b);
    printf("%d\n",c);
    return 0;
}
int sum(int a, int b){
    printf("%d\n",a);
    printf("%d\n",b);
    return a+b;
}
/* 输出：1 1 2 3 */

/* 定义局部变量时，系统不会对其初始化。定义全局变量时，系统会自动对其初始化 */
int -> 0
char -> '\0'
float -> 0
double -> 0
pointer -> NULL
```

### 数组

存储一个固定大小相同类型元素的顺序集合，由连续的内存位置组成，最低的地址对应第一个元素，最高的地址对应最后一个元素

```c
type array_name[size]; /* size>0 */
int integers[10];
int integers[5]={1,2,3,4,5}; /* 初始化 */
int integers[]={1,2,3}; /* 等价 int integers[3]={1,2,3}; 等价 int integers[3]; integers[0]=1;integers[1]=2;integers[2]=3; */

/* 多维数组 */
type arr_name[size1][size2][size3]...[sizeN];
```

### 指针

指针是一个变量，其值为另一个变量的地址，即内存位置的直接地址，一个代表内存地址的长的十六进制数

```c
type *var_name; /* 指针定义 */
int  *ptr = NULL; /* 没有确切地址时赋值NULL/编程习惯 */
int *integer_pointer; /* 整型指针 */
char *char_pointer; /* 字符型指针 */
float *float_pointer; /* 浮点型指针 */
/* `&`取地址运算符 */

#include <stdio.h>
int main(void) {
    int integer = 1;
    char *string = "STR";
    printf("value: %d, address: %p\nvalue: %s, address: %p", integer, &integer, string, &string);
    return 0;
}
/* 输出：value: 1, address: 000000000062FE4C     value: STR, address: 000000000062FE40 */
------
#include <stdio.h>
int main() {
    int integer=10;
    int *integer_pointer;
    integer_pointer=&integer;
    printf("integer: %d\n",integer);
    printf("&integer: %p\n",&integer);
    printf("pointer: %p\n",integer_pointer);
    printf("*pointer: %d\n",*integer_pointer);
    return 0;
}
/* 输出：integer: 10    &integer: 000000000062FE44    pointer: 000000000062FE44    *pointer: 10 */
------
#include <stdio.h>
int main() {
    char **s; /* s为二级指针 */
    char *p="ddd";
    s=&p; /* 赋值 */
    printf("%c\n", **s); /* 输出： d */
    printf("%s\n", *s); /* 输出： ddd */
}

char **s;
*s="ddd"; /* 出错，s未初始化，不知道内容是什么，*s会出问题 */
/* 内存指向： s:|&p| --> p:|&ddd| --> "ddd":|d|d|d| */

/* 函数指针：指向函数的指针变量，像一般函数一样用于调用函数、传递参数 */
#include <stdio.h>
int max(int x, int y) {
    return x > y ? x : y;
}
int main() {
    int (*func_pointer)(int, int) = max; /* int (*func_pointer)(int, int) = &max */
    printf("%d", func_pointer(2, 6));
}
/* 输出： 6 */

/* 回调函数：函数指针变量可以作为某个函数的参数来使用，回调函数就是一个通过函数指针调用的函数 */
#include <stdio.h>
#include <stdlib.h>
void prt_int(int (*func_p_get_int)(void)){ /* 函数作为参数 */
    for (int i = 0; i < 10; ++i) {
        printf("%d ",func_p_get_int()); /* 使用函数 */
    }
}
int func_impl(void){
    return rand();
}
int main() {
    prt_int(func_impl); /* 函数 func_impl 作为参数传入 */
}
/* 输出： 41 18467 6334 26500 19169 15724 11478 29358 26962 24464 */
```

### 字符串

实际上是使用null字符'\0'终止的一维字符数组，由于在数组的末尾存储了空字符，所以字符数组的大小比字符数多一个

```c
char str[6] = {'h', 'e', 'l', 'l', 'o', '\0'};
char str[] = "hello";
char *str = "hello";
------
#include <string.h>
strcpy(s1, s2); /* 复制字符串s2到s1，char数组，不能为char*指针，指针指向的在常量区，不能被修改 */
strcat(s1, s2); /* 连接字符串s2到s1末尾，char数组，不能为char*指针 */
strlen(s1); /* 返回字符串s1长度 */
strcmp(s1, s2); /* 比较字符串s1和s2，相同返回0，s1<s2返回小于0，s1>s2返回大于0 */
strchr(s1, ch); /* 返回一个指针，指向s1中第一次出现字符ch的位置 */
strstr(s1, s2); /* 返回一个指针，指向s1中字符串s2第一次出现的位置 */
------
#include <stdio.h>
#include <malloc.h>
#include <string.h>
int main() {
    char *p = "str1";
    char *s = "STR2";
    char *h = malloc(strlen(p) + strlen(s) + 1);
    sprintf(h,"%s%s",p,s);
    /*
     * sprintf(h,"%s",p);
     * strcat(h,s);
     */
    printf("%s", h);
}
/* 输出：str1STR2 */
```

### 结构体

```c
struct stru_name{
    member1;
    mb2;
    ...
} var_name1, va_n2 ... ;
/*
stru_name是结构体标签
member1是标准的变量定义，如int i
var_name1结构变量，定义在结构的末尾,可以指定一个或多个结构变量
*/

---1
struct Example{
    int id;
    char content[50];
    int value;
} emp_var1;
---2
struct {
    int id;
    int value;
    int etc;
} var1;
---3
struct Name{
    int id;
    int value;
    int etc;
};
struct Name var1, var2, var3[20], *var4;
/* 2和3不是一个类型，即使成员相同，*var4 = &var1 是非法的 */
---4
typedef struct {
    int id;
    int value;
    int etc;
    int other;
} Some_Name;
Some_Name var1, var2, var3[21], *var4;
---5
struct A{
    int id;
    int etc;
}
struct B{
    int id;
    struct A s_a;
}
struct C{
    int id;
    struct C *c_pointer;
}
---6
struct E; /* 对结构体B进行不完整声明 */
struct D{
    struct E *e_pointer;
    int id;
    int etc;
}
struct E{
    int id;
    int etc;
}
---
/* 访问结构成员 */
#include <stdio.h>
#include <string.h>
struct A {
    int id;
    char value[50];
};
int main() {
    struct A var_a; /* 普通定义 */
    var_a.id = 5; /* 成员访问 */
    strcpy(var_a.value, "some-value");
    printf("%d,%s\n", var_a.id, var_a.value);
    struct A *var_b; /* 指针定义 */
    var_b = &var_a; /* 指针访问 */
    var_b->id = 9;
    strcpy(var_b->value, "pointer-value");
    printf("%d,%s\n", var_b->id, var_b->value);
}
/* 输出： 5,some-value  9,pointer-value */

/*
 * 结构体中成员变量分配的空间是按照成员变量中占用空间最大的来作为分配单位
 * 成员变量的存储空间也是不能跨分配单位的,如果当前的空间不足,则会存储到下一个分配单位中
 */
#include <stdio.h>
struct A {
    char a;
    int b;
    char c;
} var_a;
struct B {
    char a;
    char b;
    int c;
} var_b;
int main() {
    printf("%lu,%lu", sizeof(var_a), sizeof(var_b));
}
/* 输出： 12,8 */
/*
var_a：a(1Byte) + 空闲(3Byte) + b(4Byte) + c(1Byte) + 空闲(3Byte) = 12Byte / 以int：4字节为单位分配空间
var_b：a(1Byte) + b(1Byte) + 空闲(2Byte) + c(4Byte) = 8Byte / 以int：4字节为单位分配空间
*/
```

### 位域

把一个字节中的二进位划分为几个不同的区域，并说明每个区域的位数，每个域有一个域名，允许在程序中按域名进行操作

- 一个位域必须存储在同一个字节中，不能跨两个字节，因此位域的长度不能大于一个字节的长度
- 如果大于计算机的整数字长，编译器可能会允许域的内存重叠，也可能会把大于一个域的部分存储在下一个字中
- 一个字节所剩空间不够存放另一位域时，应从下一单元起存放该位域
- 也可以有意使某位域从下一单元开始

```c
struct Some_Name {
    type var_name:bitlength;
    ...
    int a:8;
    int b:2;
    int c:6;
} var1;
/* var1为Some_Name变量，占2字节(16bit)，位域a占8位，b占2位，c占6位 */
------
struct Example{
    unsigned a:4;
    unsigned  :4; /* 空域，占位使得下一域从单元开始存放。无名，无法使用 */
    unsigned b:2;
    unsigned c:6;
}

/* 位域使用 */
#include <stdio.h>
struct A {
    unsigned int a:4;
    unsigned int :4;
    unsigned int b:2;
    unsigned int c:6;
};
int main() {
    struct A a; /* 普通定义 */
    a.a = 14; /* 即1110 */
    a.b = 2; /* 即10 */
    a.c = 63; /* 即111110 */
    printf("%d,%d,%d\n", a.a, a.b, a.c); /* 普通访问 */
    struct A *b; /* 指针定义 */
    b = &a;
    printf("%d,%d,%d\n", b->a, b->b, b->c); /* 指针访问 */
}
```

### 共用体

在相同的内存位置存储不同的数据类型。可定义一个多成员的共用体，任何时候只能有一个成员带有值

```c
union Some_name{
    type var_name;
} var_1;
---
union A{
    int a;
    float b;
    char c[20];
}

/* 共用体占用的内存应足够存储共用体中最大的成员 */
#include <stdio.h>
union A{
    int a;
    float b;
    char c[20];
} var_a;
int main() {
    printf("%lu", sizeof(var_a));
}
/* 输出： 20 */

/* 访问共用体成员 */
#include <stdio.h>
#include <string.h>
union A {
    int a;
    float b;
    char c[20];
} var_a, *var_p; /* 普通定义，指针定义 */
int main() {
    var_p = &var_a;
    var_a.a = 6;
    printf("%d,%d\n", var_a.a, var_p->a); /* 普通访问，指针访问 */
    var_a.b=1.2f;
    printf("%d,%d\n", var_a.a, var_p->a); /* 值被覆盖，输出错误值，同一块内存，故而同一时间只能一个成员有值 */
    printf("%f,%f\n", var_a.b, var_p->b);
    strcpy(var_a.c,"some-string");
    printf("%d,%d\n", var_a.a, var_p->a); /* 值被覆盖，输出错误值 */
    printf("%f,%f\n", var_a.b, var_p->b); /* 值被覆盖，输出错误值 */
    printf("%s,%s\n", var_a.c, var_p->c);
}
/*
输出：
6,6
1067030938,1067030938 '值被覆盖，输出错误值'
1.200000,1.200000
1701670771,1701670771 '值被覆盖，输出错误值'
70078545728475127000000.000000,70078545728475127000000.000000 '值被覆盖，输出错误值'
some-string,some-string
*/
```

### typedef

为类型取一个新的名字，使用大写字母（编程习惯）

```c
typedef unsigned int SIZE;
SIZE a, b, var_c;
---
typedef struct A{
    int id;
    int value;
    char content[50];
} NAME;
NAME var_1, var_2;
/* 与#define差异
 * typedef仅用于为类型定义符号，#define可以为类型、数值等定义符号
 * typedef由编译器执行解释，#define由预编译器处理
 */
#include <stdio.h>
#define TRUE 1
#define INT unsigned int
int main() {
    INT a = 10;
    printf("%d\n", a);
    printf("%d\n", TRUE);
}
/* 输出： 10  1 */
```

### 输入输出

```c
#include <stdio.h>
int main() {
    char a[50];
    scanf("%[^\n]", &a); /* 从stdin读取输入，根据format来格式输入，输入 a b c d 回车 */
    printf("%s", a); /* 输出到stdout，并根据提供的格式产生输出，输出 a b c d */
    scanf("%s", &a); /* 输入 a b c d 回车 */
    printf("%s", a); /* 输出 a*/
    fflush(stdin); /* 清除输入流 */
    int c = getchar(); /* 读取下一个可用的字符，并把它返回为一个整数，输入 abc ab */
    putchar(c); /* 字符输出到屏幕上，并返回相同的字符，输出 a */
    fflush(stdin); /* 清除输入流 */
    char str[100];
    gets(str);/* 从stdin读取一行到str所指向的缓冲区，直到一个终止符或EOF，输入 abc ab a */
    puts(str);/* 把字符串str和一个尾随的换行符写入到stdout，输出 abc ab a */
    fflush(stdin); /* 清除输入流 */
    char aa[5];
    fgets(aa, sizeof(aa),stdin); /* 读取字符存到以aa为起始地址的空间里，直到读完N-1个字符，或读完一行。自动在最后加'\0'，输入 abcde abc */
    fputs(aa,stdout); /* 输出 abcd */
}
```

### 文件读写

```c
/* 打开文件 */
FILE *fopen(const char *filename, const char *mode);
/*
r  打开文件读取/存在
w  打开文件写入/存在覆盖，不存在创建
a  打开文件追加/存在追加，不存在创建
r+ 打开文件读写/存在
w+ 打开文件读写/存在覆盖，不存在创建
a+ 打开文件读写/存在追加，不存在创建，从头读，从尾写
---二进制文件
rb wb ab rb+ r+b wb+ w+b ab+ a+b
*/

/* 关闭文件 */
int fclose(FILE *fp); /* 成功返回0，失败EOF */

/* 写入文件 */
int fputc(int c, FILE *fp); /* 成功返回写入字符，失败EOF */
int fputs(const char *s, FILE *fp); /* 成功返回非负值，失败EOF */
int fprintf(FILE *fp, const char *format, ...);

#include <stdio.h>
int main() {
    FILE *fp = NULL;
    fp = fopen("test.txt", "w+");
    fprintf(fp, "fprintf\n");
    fputs("fputs\n", fp);
    fputc('n',fp);
    fclose(fp);
}
/*
文件内容：
fprintf
fputs
n
*/

/* 读取文件 */
int fgetc(FILE *fp); /* 成功返回字符，失败EOF */
char *fgets(char *buf, int n, FILE *fp); /* 读取n-1字符，复制到buf，并追加中止字符。若\n或EOF，返回读到的字符(包括\n) */
int fscanf(FILE *fp, const char *format, ...); /* 遇到第一个空格时停止读取 */

#include <stdio.h>
int main() {
    FILE *fp = NULL;
    char buf[255];
    fp = fopen("test.txt", "r");
    fscanf(fp, "%s", buf);
    printf("%s\n", buf);
    fgets(buf, 255, fp);
    printf("%s\n", buf);
    char c = (char) fgetc(fp);
    printf("%c\n", c);
    fclose(fp);
}
/*
文件内容：
line1 1
line2 2
line3 3
输出：
line1
 1
'多一行，fgets返回了\n'
l
*/

/* 二进制读写 */
size_t fread(void *ptr, size_t size, size_t count, FILE *stream);
/* ptr保存结果指针，size数据类型大小，count数据个数，stream文件指针；返回读取数据个数 */
size_t fwrite (const void *ptr, size_t size, size_t count, FILE *stream);
/* ptr保存数据指针，size数据类型大小，count数据个数，stream文件指针；返回写入数据个数 */

#include <stdio.h>
struct A {
    int i;
    char c;
} p, p2;
int main() {
    FILE *fp = fopen("test.txt", "wb");
    p.i = 5;
    p.c = 'd';
    fwrite(&p, sizeof(p), 1, fp); /* 二进制写 */
    fclose(fp);
    FILE *fp2 = fopen("test.txt", "rb");
    fread(&p2, sizeof(p2), 1, fp2); /* 二进制读 */
    printf("%d,%c", p2.i, p2.c);
    fclose(fp2);
}

int fseek(FILE *fp, long offset, int whence);
/* 设置当前读写点到offset，whence为SEEK_SET,SEEK_CUR,SEEK_END决定从文件头,当前点,文件尾计算offset */
fseek(fp, 1, SEEK_CUR); /* 后移1字节 */
fseek(fp, -1, SEEK_CUR); /* 前移1字节 */
```
