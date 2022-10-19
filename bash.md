# 笔记

## bash 语法 @bash

文档

```bash
# https://www.gnu.org/software/bash/manual
```

注释

```bash
echo # 单行注释

# 注释以字符 # 开始
```

```bash
: <<EOF
多行注释
多行注释
EOF

# 使用命令 : 配合重定向实现多行注释的效果
```

引用

```bash
# 用于删除某些字符或单词的特殊含义，包含转义符、单引号、双引号
```

转义符

```bash
\

# 保留下个字符的字面值、行尾续行、空格转义

# --
echo \$\$ # $$
```

单引号

```bash
''

# 引号间保留字符的字面值，不能嵌套

# --
echo '$$' # $$
```

双引号

```bash
""

# 引号间部分字符有特殊含义，转义后可嵌套

# --
echo "$$" # 1000
```

ANSI-C 引用

```bash
$''

# 引号间部分转义序列有特殊含义

# --
echo $' \t\n' # 变量 IFS 的默认值
echo $'\176 \x7E \u7E \U7E' # ~ ~ ~ ~
echo $'\346\210\221 \xE6\x88\x91 \u6211 \U6211' # 我 我 我 我
```

特定于语言环境的转换

```bash
$""

# 类似 gettext 的转换，相关变量 LC_MESSAGES TEXTDOMAIN TEXTDOMAINDIR

# --
# todo
```

简单命令

```bash
# 以空格分隔并由一个控制运算符终止的单词序列，第一个词通常是要执行的命令，其余词是该命令的参数
```

管道

```bash
[time [-p]] [!] cmd1 [ | or |& cmd2 ] ...

# 操作符 |& 为 2>&1 | 的简写
# 命令在 subshell 执行，若设置 shopt -s lastpipe && set +m 则最后一个命令在当前 shell 执行
# 退出状态为最后一个命令的退出状态，若设置 set -o pipefail 则为最后一个非 0 退出状态或全部成功为 0

# --
echo 0:$$; echo 1:$BASHPID >&2 | echo 2:$BASHPID >&2 # 0:1000 2:1002 1:1001
( set -o pipefail; exit 1 | exit 3 | true; echo $? ) # 3
```

命令列表

```bash
cmd1 [ && or || or ; or & cmd2 ] ...

# 操作符 && 和 || 同优先级，其次 ; 和 & 同优先级
# 以 ; 分隔的命令顺序执行
# 以 & 结尾的命令在 subshell 异步执行
# 以 && 分隔的命令仅在前一个命令成功后才会执行后一个命令
# 以 || 分隔的命令仅在前一个命令失败后才会执行后一个命令

# --
true && echo 1 # 1
false || echo 1 # 1
time ( ( sleep 1 && echo 1 ) & ( sleep 2 && echo 2 ) ) # 1 2 real:0m2s
```

### 复合命令

```bash
# 以保留字或控制运算符开始并由对应的保留字或控制运算符终止的结构体，包含循环结构、条件结构、命令分组
```

#### 循环结构

until 语句

```bash
until test-cmds; do cmds; done
# --
( a=3; until ((a < 1)); do echo $a; ((--a)); done ) # 3 2 1
```

while 语句

```bash
while test-cmds; do cmds; done
# --
( a=3; while ((a > 0)); do echo $a; ((--a)); done ) # 3 2 1
```

for 语句

```bash
for name [ [in [words ...] ] ; ] do cmds; done
# 若省略 in 则使用位置参数，等同于 in "$@"
# --
( for i in {1..6..2}; do echo $i; done ) # 1 3 5
```

```bash
for (( expr1; expr2; expr3 )); do cmds; done
# 省略表达式等同于计算结果为 1
# --
( for (( i = 0; i < 3; ++i )); do echo $i; done ) # 0 1 2
( for (( ; ; )); do echo 1; done ) # 无限循环
```

#### 条件结构

if 语句

```bash
if test-cmds; then cmds; [elif test-cmds; then cmds;] ... [else cmds;] fi
# --
if [ "" ]; then echo 1; elif [[ "" ]]; then echo 2; elif ((0)); then echo 3; else echo 4; fi # 4
```

case 语句

```bash
case word in [ [(] pattern [| pattern] ...) cmds [;; or ;& or ;;&] ] ... esac
# 语句中 word 和 pattern 会执行扩展
# 每个子句必须以 ;; 或 ;& 或 ;;& 结尾
# 若 pattern 为 * 则类似 default 语句
# 使用 ;; 则在第一个匹配后结束，使用 ;& 则类似贯穿继续执行命令，使用 ;;& 则继续下一个匹配测试
# 若设置 shopt -s nocasematch 则匹配忽略大小写
# --
( a=1; case $a in [[:digit:]]*) echo num ;; *) echo other ;; esac ) # num
```

select 语句

```bash
select name [in words ...]; do cmds; done
# 若省略 in 则使用位置参数，等同于 in "$@"
# 输出 $PS3 并读取一行标准输入，若有对应数字则 name 设置为对应单词并执行
# 若读取空行则再次输出提示，若读取 <EOF> 则结束，若读取其他值则 name 为空
# 读取的行保存在变量 REPLY 中
# --
( select i in `ls`; do echo $i; break; done ) # 1) anaconda-ks.cfg 2) file
```

计算算术表达式

```bash
(( expression ))
# 等同于 let "expression"
# 表达式的值非 0 则为真
# --
(( 0 )) && echo "$?:T" || echo "$?:F" # 1:F
(( 3 )) && echo "$?:T" || echo "$?:F" # 0:T
```

计算条件表达式

```bash
[[ expression ]]
# 操作符 = 或 == 或 != 右侧字符串将被视为模式匹配，操作符 = 等同于 ==
# 操作符 =~ 右侧字符串将被视为扩展正则表达式
# 若设置 shopt -s nocasematch 则忽略大小写
# 若模式匹配保存于变量中且被引号括起则展开后将被视为字符串
# 若 && 或 || 的 condition1 就能确定整个表达式的返回值则不会计算 condition2
# --
( a='[[:digit:]]*'; [[ 333 == $a ]] && echo "$?:T" || echo "$?:F" ) # 0:T
( a='[[:digit:]]*'; [[ '[[:digit:]]*' == "$a" ]] && echo "$?:T" || echo "$?:F" ) # 0:T
[[ "some @_* str" =~ ^[[:alnum:][:punct:][:blank:]]+$ ]] && echo "$?:T" || echo "$?:F" # 0:T
```

#### 命令分组

圆括号命令分组

```bash
( cmds )
# 命令在 subshell 执行
# --
echo $$; (echo $BASH_SUBSHELL; echo $$; echo $BASHPID) # 1000 1 1000 1001
```

花括号命令分组

```bash
{ cmds; }
# 命令在当前 shell 执行
# 花括号是关键字因此需要空格、分号等元字符与命令隔开，而圆括号是运算符因此可以不用隔开
# --
echo $$; { echo $BASH_SUBSHELL; echo $$; echo $BASHPID; } # 1000 0 1000 1000
```

### 条件表达式

```bash
[ condition ]
test "condition"
[[ condition ]]
# 在 [[ ]] 中 < 和 > 运算符使用当前语言环境的字典顺序排序，在 test 命令中使用 ASCII 排序
-a # 文件存在则为真
-b # 文件存在且为块设备则为真
-c # 文件存在且为字符设备则为真
-d # 文件存在且为目录则为真
-e # 文件存在则为真
-f # 文件存在且为普通文件则为真
-g # 文件存在且设置了 sgid 则为真
-h # 文件存在且为链接文件则为真
-k # 文件存在且设置了 sticky 则为真
-p # 文件存在且为命名管道则为真
-r # 文件存在且为可读的则为真
-s # 文件存在且大小大于 0 则为真
-t # 描述符已打开且指向终端则为真
-u # 文件存在且设置了 suid 则为真
-w # 文件存在且为可写的则为真
-x # 文件存在且为可执行的则为真
-G # 文件存在且所有者为 egid 则为真
-L # 文件存在且为链接文件则为真
-N # 文件存在且自上次读取后已被修改则为真
-O # 文件存在且所有者为 euid 则为真
-S # 文件存在且为套接字则为真
-ef # 文件 1 和文件 2 引用相同的设备和 inode 号则为真
-nt # 文件 1 的修改日期比文件 2 新，或文件 1 存在文件 2 不存在则为真
-ot # 文件 1 的修改日期比文件 2 旧，或文件 2 存在文件 1 不存在则为真
-o # 对应 shell 属性被启用则为真
-v # 对应 shell 变量被赋值则为真
-R # 对应 shell 变量为 nameref 且被赋值则为真
-z  # 字符串的长度为 0 则为真
-n  # 字符串的长度不为 0 则为真
str # 字符串的长度不为 0 则为真
== # 字符串 1 与字符串 2 相同（在 [[ ]] 中将执行模式匹配）则为真
=~ # 字符串 1 与字符串 2 匹配（仅在 [[ ]] 中使用，字符串 2 被视为扩展正则表达式）则为真
=  # 字符串 1 与字符串 2 相同则为真，同 == 操作符
!= # 字符串 1 与字符串 2 不同则为真
< # 字典排序中字符串 1 在字符串 2 之前则为真
> # 字典排序中字符串 1 在字符串 2 之后则为真
# 下述操作符左右可为正负整数，在 [[ ]] 中可视为算术表达式
-eq # 参数 1 与参数 2 相等则为真
-ne # 参数 1 与参数 2 不等则为真
-lt # 参数 1 小于参数 2 则为真
-le # 参数 1 小等于参数 2 则为真
-gt # 参数 1 大于参数 2 则为真
-ge # 参数 1 大等于参数 2 则为真
# --
[[ -f /etc/hosts ]] && echo T # T
[[ -k /tmp ]] && echo T # T
[[ -u /bin/passwd ]] && echo T # T
[[ -g /bin/crontab ]] && echo T # T
( a='[[:digit:]]*'; [[ 333 == $a ]] && echo "$?:T" || echo "$?:F" ) # 0:T
( a='[[:digit:]]*'; [[ '[[:digit:]]*' == "$a" ]] && echo "$?:T" || echo "$?:F" ) # 0:T
[[ "some @_* str" =~ ^[[:alnum:][:punct:][:blank:]]+$ ]] && echo "$?:T" || echo "$?:F" # 0:T
[[ 3*3 -gt 2*3 ]] && echo T # T
```

### 算术表达式

```bash
let expression
declare -i var=expression
(( expression ))
$(( expression ))
# 表达式计算以固定宽度的整数进行，不进行溢出检查，但除以 0 会报错，运算符优先级与 C 语言相同
# 可以直接使用变量而无需参数展开语法，未设置或空的变量在按名称引用时为 0
# 前导 0 视为八进制数，前导 0x 或 0X 视为十六进制数，可使用 [base#]n 指定进制（进制在 2-64 之间）默认十进制
# 指定的进制需要字母时按小写、大写、字符 @ 和字符 _ 的顺序，若进制小等于 36 则可以互换大小写表示 10-35 之间的数
# 运算符按优先级计算，括号中的子表达式将优先进行计算
# 下述优先级递减
n++ n--   # 后增，后减
+ -       # 一元加，一元减
++n --n   # 先增，先减
! ~       # 逻辑操作，按位否定
**        # 幂操作
* / %     # 乘法，除法，取余
+ -       # 加法，减法
<< >>     # 左移位，右移位
<= >= < > # 比较
== !==    # 等，非等
&         # 位与
^         # 位异或
|         # 位或
&&        # 和
||        # 或
expression?expression:expression # 条件运算
= *= /= %= += -= <<= >>= &= ^= |= # 赋值语句
expression1 , expression2 # 逗号
# --
echo $(( 64#_ )) # 63
( a=1; echo $(( a++<<3 )) ) # 8
( a=1; echo $(( ++a<<3 )) ) # 16
echo $(( 2#101 )) $(( 2#101 & 2#000 )) $(( 2#101 | 2#000 )) # 5 0 5
echo $(( 0?9:3 )) # 3
(( 0 || 3 )) && echo T; (( 0 && 3 )) || echo F # T F
echo $(( 0 || 3 )) $(( 0 && 3 )) # 1 0
```

### 函数

```bash
fname () compound-cmds [redirections]
function fname [()] compound-cmds [redirections]
# 函数体可以是 { cmds; } 或 ( cmds ) 或 (( cmds )) 或 [[ cmds ]] 等
# 函数接收的参数在执行过程中成为位置参数，同时 $# 同步变化，但 $0 不变
# shell 使用动态范围控制变量可见性，因此函数看到的变量值取决于调用者看到的值
# 使用 local 创建的局部变量优先级高，局部变量仅在函数、子函数及其调用的函数中可见
# 使用 unset 时若有本地变量则删除本地变量，否则向上查找变量以进行删除
# 使用 return 结束执行并返回状态，默认返回状态为最后一个命令的退出状态
# --
( function fn() { pwd; } >&2; fn >/dev/null ) # /root
( a=1; fn(){ local a=2; echo $a; }; fn; echo $a ) # 2 1
( a=1; fn(){ local a=2; unset a; echo "${a-unset}"; }; fn; echo $a ) # unset 1
( a=1; fn(){ unset a; echo "${a-unset}"; }; fn; echo "${a-unset}" ) # unset unset
```

### 参数

```bash
# 参数是存储值的实体，可以是名称、数字或特定字符
```

参数操作

```bash
name=[value]
# 变量赋值，语句中等号的左右不能有空格
# 语句中 value 会执行扩展，若省略则变量被分配空字符串
# 若参数已被赋值（包括空字符串）则该参数已被设置 (set)，设置变量后可使用 unset 命令取消设置 (unset)
# --
( declare a; echo ${a-unset}; a=1; echo ${a-unset} ) # unset 1


$name
${name}
# 获取变量的值，使用 ${name} 可避免紧跟其后的字符串被视为变量名的一部分
# --
( a=1; echo ${a}str ) # 1str
```

位置参数

```bash
$N
${N}
# 由一位或多位数字表示的参数，即 $1 .. $N 但不含 $0
# N 为一位数时可使用 $N 引用，为多位数时须使用 ${N} 引用
# --
( set -- a1 a2 a3; echo $2 ) # a2
```

特殊参数

```bash
$*
$@
# 扩展为所有位置参数
# 对于 $* 无引号括起则为默认分词，有引号括起则格式为 "arg1${IFS:0:1}arg2${IFS:0:1}arg3"
# 对于 $@ 无引号括起则为默认分词，有引号括起则格式为 "arg1" "arg2" "arg3"
# --
( IFS=$'@\t \n'; set -- 'a 1' a2 a3; echo $* . "$*" ) # "a" "1" "a2" "a3" . "a 1@a2@a3"
( IFS=$'@\t \n'; set -- 'a 1' a2 a3; echo $@ . "$@" ) # "a" "1" "a2" "a3" . "a 1" "a2" "a3"


$#
# 位置参数的总数


${#N}
# 第 N 个位置参数值的长度


$?
# 最近执行的前台命令的退出状态，正常退出的状态为 0


$-
# 调用 shell 时指定的选项
# --
echo $- # himBHs


$$
# 当前 shell 的 pid 值
# 在 subshell 中其值为调用者的 pid 值，若要获取 subshell 的 pid 应使用 $BASHPID


$!
# 最近执行的后台任务的 pid 值


$0
# 扩展为 shell 或 shell 脚本的名称
# --
echo $0; bash -c 'echo $0'; bash -c 'echo $0' '$0'; /bin/bash -c 'echo $0' # -bash bash $0 /bin/bash
```

### 变量

```bash
# 变量是由名称表示的参数，有一个值和零或多个属性
```

同 sh 中的变量

```bash
$HOME
# 当前用户的家目录


$IFS
# 分词时使用的字符列表
# 默认值为 <space><tab><newline>


$PATH
# 以冒号分隔的目录列表，命令的寻找路径
# 零长度目录表示当前目录，类似 "<path>::<path>" "<path>:" ":<path>"


$PS1
# 主要提示字符串，类似 "user@host:~$ "


$PS2
# 辅助提示字符串，续行或多行输入的提示符，类似 "> "
```

bash 特有的变量

```bash
$BASHPID
# 当前 shell 的 pid 值，与 $$ 的不同在于 $$ 在 subshell 中不会被重新初始化而是继承父级的值


$BASH_COMMAND
# 当前或即将执行的命令
# 若是因 traps 而执行的则显示为触发时正在执行的命令


$BASH_SOURCE
# 数组，其成员为数组 $FUNCNAME 定义的函数的源文件，类似文件调用堆栈
# 其中 ${BASH_SOURCE[0]} 为当前脚本名称
# 函数 ${FUNCNAME[i]} 定义于 ${BASH_SOURCE[i]} 且被 ${BASH_SOURCE[i+1]} 调用


$BASH_SUBSHELL
# 初始值为 0 每当进入 subshell 环境时增加 1


$COMP_CWORD
# 当前光标位置的单词在 $COMP_WORDS 中的索引


$COMP_LINE
# 当前命令行输入的字符


$COMP_POINT
# 光标在当前命令行的位置，若光标在末尾则等于 ${#COMP_LINE}


$COMP_WORDBREAKS
# 单词分隔符


$COMP_WORDS
# 数组，当前命令行输入的所有单词


$COMPREPLY
# 数组，所有的补全候选词


$FIGNORE
# 文件名补全时要忽略的后缀，以冒号分隔


$FUNCNAME
# 数组，其成员为当前调用堆栈中的所有函数，类似函数调用堆栈
# 变量仅在函数执行时存在
# 其中 ${FUNCNAME[0]} 为当前执行的函数名，最后一个元素 ${FUNCNAME[-1]} 为 main
# 函数 ${FUNCNAME[i]} 在文件 ${BASH_SOURCE[i+1]} 在行号 ${BASH_LINENO[i]} 被调用


$FUNCNEST
# 若大于 0 则为函数最大嵌套等级，超过限制会使得整个命令终止


$HISTCONTROL
# 以冒号分隔的值，设置命令如何保存在历史记录中
# 不管该变量如何设置，多行命令始终会添加到历史记录中
ignorespace # 以空格开头的行不会保存
ignoredups # 与上个历史条目匹配的行不会保存
ignoreboth # 为 ignorespace 和 ignoredups 的简写
erasedups # 保存该行之前从历史记录中删除与该行匹配的所有行


$HISTFILE
# 命令历史记录保存位置，默认 ~/.bash_history


$HISTSIZE
# 命令历史记录保存条数，默认 500


$PPID
# 父进程的 pid 值


$PS0
# 在读取命令后且执行命令前扩展并显示


$PS3
# 命令 select 的输入提示字符串


$PS4
# 执行跟踪期间在命令前显示，第一个字符将复制多次以体现级别，类似 set -x 时的 "+ "


$RANDOM
# 扩展为一个 0-32767 的随机数
# 对其进行赋值可以指定 seed 以重复生成相同随机数序列


$SHLVL
# 每启动一个新的 bash 实例时增加 1


$SRANDOM
# 扩展为一个 32bit 的伪随机数
# 不能指定 seed


$TMOUT
# 若大于 0 则为默认超时设置
# 命令等待终端输入超时则终止，交互式 shell 等待完整输入行超时则退出
```

### 数组

```bash
# bash 支持一维索引数组和关联数组
# 数组大小没有限制，也不要求成员索引连续或成员分配连续
# 索引数组使用整数引用（包括算术表达式）并且从零开始，关联数组使用任意字符串引用
# 命令 declare local readonly 都可以使用 -a 创建索引数组，使用 -A 创建关联数组
```

索引数组

```bash
name[subscript]=value
name=(value1 value2 ...)
declare -a name
declare -a name[subscript]=value
declare -a name=(value1 value2 ...)
# subscript 为数字或计算结果为数字的算术表达式
# 若提供了可选的下标则赋值给该索引，否则索引是数组中最后一个索引加一，索引默认从零开始
# 索引左往右从 0 计，右往左从 -1 计，即通常 0 为第一个元素而 -1 为最后一个元素
# --
( a=(e0 e1 e2); echo ${a[@]@K} ) # 0 "e0" 1 "e1" 2 "e2"
( a[0]=e0; declare -a b[3]=e3; echo ${a[@]@K} . ${b[@]@K} ) # 0 "e0" . 3 "e3"
```

关联数组

```bash
declare -A name
declare -A name[subscript]=value
declare -A name=(key1 value1 key2 value2 ...)
declare -A name=([key1]=value1 [key2]=value2 ...)
# 关联数组以哈希顺序存储，遍历顺序不一定与输入顺序相同，可使用索引数组记录需要的顺序
# --
( declare -A a=(k1 v1 k2 v2); echo ${a[@]@K} ) # k1 "v1" k2 "v2"
( declare -A a=([k1]=v1 [k2]=v2); echo ${a[@]@K} ) # k1 "v1" k2 "v2"
( declare -A a=([a]=e1 [b]=e2); IFS=@; echo "${!a[@]}" . "${!a[*]}" ) # b a . b@a
```

数组操作

```bash
${arr[idx]}
# 引用对应数组元素
# 索引左往右从 0 计，右往左从 -1 计，即通常 0 为第一个元素而 -1 为最后一个元素
# 若下标为 @ 或 * 则扩展为数组所有元素
# --
( a=(e0 e1 e2 e3); echo ${a[1]} ${a[-1]} ) # e1 e3


${arr[@]}
${arr[*]}
# 获取数组内的所有元素值
# 两者区别参考变量 $@ 与 $* 的区别
# --
( a=(e0 e1 e2 e3); for i in "${a[@]}"; do echo $i.L; done ) # e0.L e1.L e2.L e3.L
( a=(e0 e1 e2 e3); for i in "${a[*]}"; do echo $i.L; done ) # e0 e1 e2 e3.L


${arr}
# 与 ${arr[0]} 等价，获取数组第一个元素
# --
( a=(e0 e1 e2); echo ${a} $a ) # e0 e0


${#arr[idx]}
# 获取数组索引为 idx 的元素值长度，即元素 ${arr[idx]} 的长度
# 若下标为 @ 或 * 则扩展为数组自身的长度
# --
( a=(ele0 ele11 ele222); echo ${#a[@]} ${#a[1]} ) # 3 5


${#arr[@]}
${#arr[*]}
# 获取数组自身的长度
# --
( a=(ele0 ele11 ele222); echo ${#a[@]} ${#a[1]} ) # 3 5


${!arr[@]}
${!arr[*]}
# 获取数组所有元素的索引名
# 两者区别参考变量 $@ 与 $* 的区别
# --
( a=(e0 e1); declare -A b=(k1 v1 k2 v2); echo ${!a[@]} . ${!b[@]} ) # 0 1 . k1 k2


unset arr[idx]
# 删除指定索引的元素，删除最后一个元素并不会使得数组被删除
# --
( a=(e0 e1 e2); unset a[1]; echo ${a[@]@K} ) # 0 "e0" 2 "e2"


unset arr
unset arr[@]
unset arr[*]
# 删除整个数组
# --
( a=(e0 e1 e2); unset a; echo ${a-unset} ) # unset
( a=(e0 e1 e2); unset a[@]; echo ${a-unset} ) # unset


arr+=(value1 value2 ...)
# 索引数组可使用 += 添加元素
# --
( a=(e0); a+=(e1); b[5]=e5; b+=(e6); echo ${a[@]@K} . ${b[@]@K} ) # 0 "e0" 1 "e1" . 5 "e5" 6 "e6"


arr+=(key1 value1 key2 value2 ...)
arr+=([key1]=value1 [key2]=value2 ...)
# 关联数组可使用 += 添加元素
# --
( declare -A a=(k1 v1 k2 v2); a+=(k3 v3); echo ${a[@]@K} ) # k1 "v1" k2 "v2" k3 "v3"
( declare -A a=([k1]=v1 [k2]=v2); a+=([k3]=v3); echo ${a[@]@K} ) # k1 "v1" k2 "v2" k3 "v3"
```

### 扩展

```bash
# 包含花括号扩展、波浪号扩展、参数和变量扩展、命令替换、算术扩展、分词、文件名扩展
# 顺序为花括号扩展；波浪号扩展，参数和变量扩展，算术扩展，命令替换；分词；文件名扩展
# 若系统支持进程替换，则与波浪号扩展，参数和变量扩展，算术扩展，命令替换同时进行
# 只有花括号扩展、分词、文件名扩展会增加单词数，其他只会一个词扩展为一个词，但 "$@" $* "${name[@]}" ${name[*]} 例外
# 扩展后会进行引用删除，删除原词中存在的引用符，除非其引用了自身
```

花括号扩展

```bash
# 可生成任意字符串，可嵌套展开，保留原始顺序
# 以 ${ 开始则不视为花括号扩展，且在结束的 } 前禁止花括号扩展


{a,b,c}
# 扩展为 a b c 三个字符串
# --
echo /usr/{bin,lib,src} # /usr/bin /usr/lib /usr/src
echo a{b{1..6..2},c} # ab1 ab3 ab5 ac


{x..y..step}
# 在 x-y 之间按步进 step 进行扩展，语句中 x 与 y 必须是相同类型
# 若 step 省略则默认为 1 或 -1
# 数字可添加前缀 0 以保持相同宽度
# --
echo {1..3} . {1..6..2} . {6..1..-2} . {a..c} # 1 2 3 . 1 3 5 . 6 4 2 . a b c
echo {01..100..2} # 001 003 005 007 .. 099
echo {00001..100..2} # 00001 00003 00005 00007 .. 00099
```

波浪号扩展

```bash
# 以 ~ 开始的字符到 / 为止（若有）的所有字符被视为可能的登录名并扩展为该用户的家目录
# 每个变量赋值都会在 : 或第一个 = 之后立即检查是否有波浪号扩展


~[user]
# 扩展为 user 用户的家目录，若省略 user 则为当前用户家目录
# --
echo ~ ~bin ~daemon ~null # /root /bin /sbin ~null
( a=~root:~bin:~daemon; echo $a; b=~root/=~bin; echo $b ) # /root:/bin:/sbin /root/=~bin


~+
# 扩展为 $PWD
# --
( cd; cd /tmp; echo ~+ ) # /tmp


~-
# 扩展为 $OLDPWD
# --
( cd; cd /tmp; echo ~- ) # /root


~[+-][N]
# 扩展为 $DIRSTACK 中对应的元素，可使用 dirs [+-N] 查看
# --
( cd; pushd /sbin; pushd /tmp; pushd /bin; echo ~0 ~+1 ~3 ~-0 ) # /bin /tmp /root /root
```

参数和变量扩展

```bash
# 参数名可用可选的花括号括起以避免变量名被其后字符影响
# 语句中 word 会执行波浪号扩展、参数和变量扩展、命令替换、算术扩展


$name
${name}
# 获取变量 name 的值


${!name}
# 若变量不是 nameref 则为间接扩展，将变量的值作为变量名，而后扩展该变量名
# 若变量是 nameref 则不执行完整的间接扩展，而是将其扩展为引用对象的名称
# --
( nameA=valueA; declare -n namerefB=nameA; nameC=nameA; echo ${!nameC} ${!namerefB} ) # valueA nameA


${name-word}
${name:-word}
# 对于 - 若 name 未设置，返回 word 且 name 不变
# 对于 :- 若 name 为空或未设置，返回 word 且 name 不变


${name=word}
${name:=word}
# 对于 = 若 name 未设置，返回 word 且 name 赋值为 word
# 对于 := 若 name 为空或未设置，返回 word 且 name 赋值为 word


${name?word}
${name:?word}
# 对于 ? 若 name 未设置，报错且报错信息为 word
# 对于 :? 若 name 为空或未设置，报错且报错信息为 word
# --
( a=''; echo ${a:?invalid} ) # -bash: a: invalid
echo ${a?} # -bash: a: parameter null or not set


${name+word}
${name:+word}
# 对于 + 若 name 未设置，返回空，否则返回 word 且 name 不变
# 对于 :+ 若 name 为空或未设置，返回空，否则返回 word 且 name 不变
# --
( a=''; echo "${a:+notice}" . ${a:=$(pwd)} ${a:+string} ) # "" . /root string


${name:offset}
${name:offset:length}
# 子字符串扩展
# 从 offset 起截取 length 个字符作为返回值且 name 不变，省略 length 截取至结尾，且 offset 和 length 可为算术表达式
# offset 左往右从 0 计，右往左从 -1 计，而 length 正数为长度，负数为距离末尾的偏移量
# offset 负数时需添加空格或括号 ${name: -offset:length} 或 ${name:(-offset):length} 以避免与 ${name:-word} 混淆
# 若 name 为 @ 或 * 则截取位置参数列表，此时 offset 同上，而 length 须大于 0
# 若 name 为数组且下标为 [@] 或 [*] 则截取数组元素列表，此时 offset 同上，而 length 须大于 0
# --
( a=123456; echo ${a:${#a}-4:4} ) # 3456
( a=a1b2c3d4; echo ${a:2:3} ${a: -5:3} ${a:2:-2} ${a: -5:-1} ) # b2c 2c3 b2c3 2c3d


${!prefix@}
${!prefix*}
# 获取以 prefix 开头的变量名
# --
( a_1=1; a_2=2; a_3=3; IFS=@; echo "${!a_*}" . "${!a_@}" ) # a_1@a_2@a_3 . a_1 a_2 a_3


${!name[@]}
${!name[*]}
# 若 name 是数组则扩展为数组索引列表，若不是数组且已被设置则为 0 否则为空
# --
( declare -A a=([a]=1 [b]=2); echo ${!a[@]} ) # a b


${#name}
# 变量 name 的值的长度
# 若 name 为 @ 或 * 时为位置参数总数
# 若 name 为数字时为对应位置参数其值的长度
# 若 name 为数组且下标为 [@] 或 [*] 时为数组元素总数
# 若 name 为数组且下标为数字时为对应数组元素其值的长度


${name#word}
${name##word}
# 删除 name 值中符合的前缀（必须从第一个字符开始就匹配）并返回结果，且 name 不变
# 语句中 # 为最短匹配，而 ## 为最长匹配
# 若 name 为 @ 或 * 时操作将依次应用于位置参数，若为数组且下标为 [@] 或 [*] 时操作将依次应用于数组元素
# --
( a=a1a1b2; echo ${a#*a1} ${a##*a1} ) # a1b2 b2


${name%word}
${name%%word}
# 删除 name 值中符合的后缀（必须从最后一个字符开始就匹配）并返回结果，且 name 不变
# 语句中 % 为最短匹配，而 %% 为最长匹配
# 若 name 为 @ 或 * 时操作将依次应用于位置参数，若为数组且下标为 [@] 或 [*] 时操作将依次应用于数组元素
# --
( a=a1b2b2; echo ${a%b2*} ${a%%b2*} ) # a1b2 a1


${name/pattern/str}
# 符合 pattern 的最长匹配将被替换为 str
# 若 str 为空则删除对应匹配，此时可省略 /str
# 若设置 shopt -s nocasematch 则匹配忽略大小写
# 若 pattern 以 / 开头则替换所有匹配，否则只替换第一个匹配
# 若 pattern 以 # 开头则必须从第一个字符开始就匹配，若以 % 开头则必须从最后一个字符开始就匹配
# 若 name 为 @ 或 * 时操作将依次应用于位置参数，若为数组且下标为 [@] 或 [*] 时操作将依次应用于数组元素
# --
( n=ababc; echo ${n/a*a} ${n/ab/C} ${n//ab/C} ${n/#*b/C} ${n/%b*/C} ) # bc Cabc CCc Cc aC


${name^pattern}
${name^^pattern}
# 符合 pattern 的匹配将被替换为大写
# 语句中 ^ 替换第一个匹配，而 ^^ 替换所有匹配
# pattern 最多只匹配一个字符，若省略则类似 ? 匹配任意字符
# 若 name 为 @ 或 * 时操作将依次应用于位置参数，若为数组且下标为 [@] 或 [*] 时操作将依次应用于数组元素
# --
( n=aab; echo ${n^a} ${n^^a} ${n^} ${n^^} ) # Aab AAb Aab AAB


${name,pattern}
${name,,pattern}
# 符合 pattern 的匹配将被替换为小写
# 语句中 , 替换第一个匹配，而 ,, 替换所有匹配
# pattern 最多只匹配一个字符，若省略则类似 ? 匹配任意字符
# 若 name 为 @ 或 * 时操作将依次应用于位置参数，若为数组且下标为 [@] 或 [*] 时操作将依次应用于数组元素
# --
( n=AAB; echo ${n,A} ${n,,A} ${n,} ${n,,} ) # aAB aaB aAB aab


${name@operator}
# 依据 operator 进行对应操作
# 若 name 为 @ 或 * 时操作将依次应用于位置参数，若为数组且下标为 [@] 或 [*] 时操作将依次应用于数组元素
U # 小写转大写
u # 首字符大写
L # 大写转小写
Q # 可直接用于输入的格式
E # 同 $'' 一样进行扩展
P # 同提示字符串一样进行扩展
A # 为赋值或声明语句，执行该语句可以重建其属性和值
K # 对于索引数组或关联数组，若下标为 [@] 或 [*] 则以键值对的格式输出
a # 为参数属性标志，类似 r 表示只读
# --
( declare -r a='a"b&c'; echo ${a@Q} . ${a@E} . ${a@A} . ${a@a} ) # 'a"b&c' . a"b&c . declare -r a='a"b&c' . r
( a='\176'; echo $a ${a@E} ) # \176 ~
( a='second: \D{%S}'; echo ${a@Q} . ${a@P} ) # 'second: \D{%S}' . second: 30
( declare -A a=([a]=1 [b]=2); echo ${a[@]@K} ) # b "2" a "1"
```

命令替换

```bash
`cmds`
$(cmds)
# 在 subshell 执行 cmds 并将其替换为输出
# 使用 $() 时可以直接进行嵌套，使用 ` 则需要转义
# 使用 $() 时内部所有字符都视为命令的一部分，不做特殊处理
# $(cat file) 可换成等效且更快的 $(< file)
# --
echo "path: $(pwd)" # path: /root
echo "1:$(echo "2:$(echo "3:str")")" # 1:2:3:str 此时内部的双引号不用转义
```

算术扩展

```bash
$(( expression ))
# 对算术表达式 expression 求值并将其替换为结果
# 扩展可以嵌套
# $[ expression ] 为已废弃的语法
# 表达式中的单词都会执行参数和变量扩展、命令替换、引用删除
# --
( a=$((6*6)); echo $a $((3*3)) ) # 36 9
echo $(( 3 || 0 )) $(( 3 && 0 )) # 1 0
```

进程替换

```bash
# 使用文件名引用进程的输入或输出
# 命令列表异步运行，其输入或输出显示为文件名，该文件名作为参数传递给其他命令
# 操作符 > 和 < 与 ( 之间不能有空格，否则会被解析为重定向


<(cmds)
# 读取该文件以获得命令的输出
# --
echo <(id) # /dev/fd/63
source <(echo 'echo $BASHPID') # 1000


>(cmds)
# 写入该文件以提供命令的输入
# --
{ echo msg; echo err >&2; } 2> >(tee /tmp/err) # msg err 文件 /tmp/err 内容 err
```

分词

```bash
# 未被双引号括起的参数和变量扩展、命令替换、算术扩展的结果将执行分词
# 将 $IFS 的每个字符视为分隔符对扩展结果进行拆分，其默认值为 <space><tab><newline> 即 $' \t\n'
# 若 $IFS 未设置或为默认值则头尾的 <space><tab><newline> 序列会被忽略，其余位置的相应字符用于分词
# 若 $IFS 为其他值但含有空白字符 <space><tab><newline> 则头尾对应的 <space><tab><newline> 序列会被忽略
# 在 $IFS 中非空白字符的其他字符、其他字符同相邻空白字符、相邻空白字符序列都会分隔一个字段
# 若 $IFS 值为空则不会进行分词
# 若没有发生扩展则不会进行分词
# 显式空参数 "" 或 '' 作为空字符串传递给命令
# 扩展后是空值的无引号隐式空参数会被删除，若加上双引号则会保留并作为空字符串传递
# 空参数作为非空扩展的一部分时会被删除，即 -d"" 视为 -d
# --
( IFS=''; a='a@1 a@2 a@3'; set -- $a; echo $# "${@@Q}" ) # 1 'a@1 a@2 a@3'
( IFS='@'; a='a@1 a@2 a@3'; set -- $a; echo $# "${@@Q}" ) # 4 'a' '1 a' '2 a' '3'
( IFS='@'; set -- $(echo "a@"{1..3}); echo $# "${@@Q}" ) # 4 'a' '1 a' '2 a' '3'
( IFS='@ '; a='  a@1  a@2 a@3 '; set -- $a; echo $# "${@@Q}" ) # 6 'a' '1' 'a' '2' 'a' '3'
( IFS='@ '; a=' @a@ 1  a@2 a@3 '; set -- $a; echo $# "${@@Q}" ) # 7 '' 'a' '1' 'a' '2' 'a' '3'
( IFS='@ '; a=$'\ta@1 a@2 a@3'; set -- $a; echo $# "${@@Q}" ) # 6 $'\ta' '1' 'a' '2' 'a' '3'
```

文件名扩展

```bash
# 若单词含有未被引用的 * ? [ 且未设置 set -f 则该词被视为模式匹配
# 若没有匹配项则该词原样输出，若设置 shopt -s nullglob 则移除该词，若设置 shopt -s failglob 则不执行并输出错误
# 开头或斜杠后的 . 需要显式匹配，若设置 shopt -s dotglob 则不用，但目录 . 和 .. 始终需要显式匹配
# 若设置 shopt -s nocaseglob 则匹配忽略大小写
# --
( cd /tmp; echo _null_*; echo @; shopt -s nullglob; echo 1 _null_* 1 ) # _null_* @ 1 1
( cd /tmp; shopt -s failglob; echo _null_* ) # -bash: no match: _null_*
( cd /tmp; touch fa fA .fa .fA; echo *a; echo @; shopt -s nocaseglob; echo *a ) # fa @ fa fA
( cd /tmp; touch fa fA .fa .fA; echo *a; echo @; shopt -s dotglob; echo *a ) # fa @ .fa fa


*
# 匹配包括空字符串在内的任意字符串
# 若设置 shopt -s globstar 则 ** 匹配所有文件及目录和子目录，而 **/ 仅匹配目录和子目录


?
# 匹配任意一个字符


[]
# 匹配任意一个括号内的字符
# 若 x-y 则在当前语言环境的整理顺序和字符集下任何介于两者之间的字符都会被匹配
# 若 [] 中第一个字符是 ! 或 ^ 则匹配不在括号内的任意字符
# 在 [] 中可以将 - 放在第一个或最后一个来匹配 - 字符，也可以将 ] 放在第一个来匹配 ] 字符
# 在 [] 中可以使用字符类 [:class:] 进行匹配
  alnum # 字母和数字
  alpha # 字母
  ascii # ASCII 字符
  blank # 空格和 tab
  cntrl # 控制字符
  digit # 数字
  graph # 图形字符，除空格和控制字符外的任何字符
  lower # 小写字母
  print # 空格和图形字符，除控制字符外的任何字符
  punct # 标点字符，图形字符去掉字母数字
  space # 空白字符，包括空格 tab 换行 回车 换页
  upper # 大写字母
  word  # 单词字符，字母数字和下划线
  xdigit # 十六进制数字
# 若设置 shopt -s extglob 则可以使用扩展模式匹配
  ?(pattern) # 匹配 0 或 1 次
  *(pattern) # 匹配 0 或多次
  +(pattern) # 匹配 1 或多次
  @(pattern) # 匹配 1 次
  !(pattern) # 匹配除 pattern 外的其他情况
# --
( cd /tmp; touch 一fa 三fa 五fA; echo [一-十]f[aA] ) # 一fa 三fa 五fA
( cd /tmp; touch @fa ]fA; echo [[:punct:]]f[aA] ) # @fa ]fA
( shopt -s extglob; echo /usr/?(s)bin . /usr/!(s)bin ) # /usr/bin /usr/sbin . /usr/bin
```

引用删除

```bash
# 扩展后不是由上述扩展产生的所有不带引号的 \ ' " 都将被删除
```

重定向

```bash
# 执行命令前重定向其输入输出，按从左到右的顺序处理
# 使用 exec {name} 的语法将由 shell 分配大等于 10 的文件描述符并赋值给 name 且该重定向在命令范围外依旧存在
# 语句中 word 会执行花括号扩展、波浪号扩展、参数和变量扩展、命令替换、算术扩展、引用删除、文件名扩展和分词
# 若 word 扩展的结果超过一个单词则会报错


[n]<word
# 重定向输入
# --
( echo msg >/tmp/file; cat </tmp/file ) # msg
( exec 11</tmp/file; ll /dev/fd/11 ) # lr-x------. /dev/fd/11 -> /tmp/file
( exec {name}</tmp/file; ll /dev/fd/$name ) # lr-x------. /dev/fd/10 -> /tmp/file


[n]>[|]word
# 重定向输出
# 若设置 set -o noclobber 则使用 > 重定向至已存在的普通文件会失败，可使用 >| 进行重定向
# --
( echo msg >/tmp/file; cat </tmp/file ) # msg
( exec 11>/tmp/file; ll /dev/fd/11 ) # l-wx------. /dev/fd/11 -> /tmp/file
( exec {name}>/tmp/file; ll /dev/fd/$name ) # l-wx------. /dev/fd/10 -> /tmp/file


[n]>>word
# 追加模式重定向输出
# --
( echo msg >/tmp/file; echo msg2 >>/tmp/file; cat </tmp/file ) # msg msg2
( exec {name}>>/tmp/file; ll /dev/fd/$name ) # l-wx------. /dev/fd/10 -> /tmp/file


&>word
>&word
>word 2>&1
# 重定向标准输出和标准错误，推荐使用 &> 语法
# --
( { echo msg; echo err >&2; } &>/tmp/file; cat /tmp/file ) # msg err


&>>word
>>word 2>&1
# 追加模式重定向标准输出和标准错误


[n]<&word
# 将 word 文件描述符复制到 n
# 若省略 n 则默认为标准输入 0
# 若 word 为 - 则关闭文件描述符 n
# --
( exec 11</tmp/file; exec 22<&11; ll /dev/fd/{11,22} ) # lr-x------. 11,22->/tmp/file
( exec 11</tmp/file; exec {name}<&11; ll /dev/fd/{11,$name} ) # lr-x------. 11,10->/tmp/file


[n]>&word
# 将 word 文件描述符复制到 n
# 若省略 n 则默认为标准输出 1
# 若 word 为 - 则关闭文件描述符 n
# --
( exec 11>/tmp/file; exec 22>&11; ll /dev/fd/{11,22} ) # l-wx------. 11,22->/tmp/file
( exec 11>/tmp/file; exec {name}>&11; ll /dev/fd/{11,$name} ) # l-wx------. 11,10->/tmp/file


[n]<&word-
# 将 word 文件描述符复制到 n 之后关闭 n
# 若省略 n 则默认为标准输入 0
# --
( exec 11</tmp/file; exec 22<&11-; ll /dev/fd/{11,22} ) # error:11, lr-x------. 22->/tmp/file


[n]>&word-
# 将 word 文件描述符复制到 n 之后关闭 n
# 若省略 n 则默认为标准输出 1
# --
( exec 11>/tmp/file; exec 22>&11-; ll /dev/fd/{11,22} ) # error:11, l-wx------. 22->/tmp/file


[n]<>word
# 使用文件描述符 n 进行读取和写入
# --
( exec 11<>/tmp/file; ll /dev/fd/11 ) # lrwx------. /dev/fd/11 -> /tmp/file
```

内嵌文档

```bash
[n]<<[-]word
文档内容
delimiter
# Here Documents
# 读取输入直到出现只有指定结束符的行（不能有空格），其间读取的所有行作为命令的标准输入
# word 不执行参数和变量扩展、命令替换、算术扩展、文件名扩展
# 若 word 被引用，则 delimiter 是 word 引用删除的结果，并且读取的内容不展开
# 若 word 未被引用，则读取的内容会执行参数和变量扩展、命令替换、算术扩展
# 若操作符是 <<- 则输入行和结束行的前导 tab 序列会被删除，可用于脚本的代码对齐，仅对 tab 有效，空格不生效
# --
cat <<EOF
$(pwd) # 输出 /root
EOF
cat <<\EOF
$(pwd) # 输出 $(pwd)
EOF
```

内嵌字符串

```bash
[n]<<<word
# Here Strings
# Here Documents 的变体，将 word 扩展后以一个字符串并附加一个换行的形式作为命令的标准输入
# word 会执行波浪号扩展、参数和变量扩展、命令替换、算术扩展、引用删除，但不执行文件名扩展和分词
# --
cat <<<~ # /root
( read a <<<'str'; echo $a ) # str
```

作业

```bash
# jobs 命令用于查看当前的作业列表
# 使用 ^Z 即 Ctrl + Z 使得进程立即后台挂起并返回 bash 同时待处理的输出和预输入被丢弃
# 使用 %n 引用指定作业，使用 %% 或 %+ 或 % 引用当前作业，使用 %- 引用前一个作业，只有一个作业则 %+ 和 %- 都为该作业
# 用命令的前缀来引用作业，使用 %ce 引用以 ce 开头的作业，使用 %?ce 引用包含 ce 的作业，若有多个匹配则报错
# 作业的输出中（如 jobs 的输出）当前作业始终标记为 + 前一个作业标记为 -
# 使用缩写 %1 等同于 fg %1 将工作 1 带到前台，而 %1 & 等同于 bg %1 在后台恢复作业 1
# 作业更改状态时 shell 为避免中断其他的输出 bash 会在其即将输出提示时才报告作业状态的变化
# 若设置 set -b 则会立即报告作业状态的变化
# wait 命令在作业更改状态时将返回，使用 -f 选项则会在作业或进程终止后返回
```

别名

```bash
# 可使用 alias 和 unalias 命令设置和取消别名
# 替换文本可包含任何有效的 shell 输入，包括 shell 元字符
# 若命令的第一个词没有括起或转义，则会检查其是否有别名，若有则替换为设置的别名值
# 替换后的别名值的第一个词会再次检查是否有别名，若该词与正在扩展的别名相同则不会再次扩展
# 若替换后的别名值的最后字符是空字符，则会检查其后的下个词是否有别名
# bash 执行输入命令前至少读取一行完整输入，别名在读取时展开而非执行时，因此同一行命令不受新别名影响，别名在下一行失效
# 函数同理，别名在读取函数定义时扩展而不是执行时，函数中定义的别名在函数执行前不可用，函数定义后新别名在函数中不生效
# 没有使用参数的机制，若需要参数则应使用函数
# 非交互式 shell 不会展开别名，除非设置 shopt -s expand_aliases
# --
( alias echo='echo 1'; eval echo ) # 1 设置别名后执行 echo 即为执行 echo 1 单词相同因此不会递归展开
( a(){ echo; }; alias echo='echo 1'; eval 'a; echo' ) # "" 1 后定义的别名不在函数生效
( alias echo='echo 1'; eval 'a(){ echo; }; a') # 1 先定义的别名生效
bash -c 'shopt -s expand_aliases
  alias a="echo a str"
  alias b="echo b str "
  alias c="c str "
  a c
  echo .
  b c' # a str c . b str c str 其中 c 也进行了别名扩展
```

简单命令扩展

```bash
# 若扩展后没有产生命令则变量分配 (var=xxx cmds) 影响当前 shell 环境，否则只影响命令的执行环境
# 若扩展后没有产生命令则重定向执行且不影响当前环境
# ---
bash -c $'shopt -s expand_aliases\nalias empty=""\na=1 empty\necho $a' # 1
bash -c $'shopt -s expand_aliases\na=1 :\necho ${a-unset}' # unset
```

命令查找和执行

```bash
# 若扩展后有命令和可选的参数列表则查找并执行该命令
# 命令不含斜杠 -> shell 函数 -> 内置命令 -> $PATH 可执行文件 -> command_not_found_handle 函数 -> 报错并以 127 退出
# 若命令存在但没有可执行权限则报错并以 126 退出
# bash 会记录命令的完整路径，仅在记录中找不到该命令时才会对 $PATH 中的目录进行完整搜索，可使用 hash 查看和管理
# --
( function echo(){ pwd; date +%S; builtin echo .; }; echo; hash) # /root 30 . 1 /usr/bin/date
```

命令执行环境

```bash
# 若简单命令不是内置命令或 shell 函数，则会在单独的执行环境中执行
# 在单独的执行环境中执行的命令不会影响 shell 执行环境
# 命令替换、圆括号命令分组、异步命令、参与管道的内置命令都会在 subshell 执行环境中执行
# 在 subshell 执行环境所做的更改不会影响 shell 执行环境
# 非 POSIX 模式下命令替换的 subshell 不会继承父 shell 的 set -e 选项


# shell 执行环境
# shell 在调用时继承的打开的文件，以及 exec 修改的重定向
# cd 或 pushd 或 popd 设置的，或 shell 在调用时继承的当前工作目录
# umask 设置的，或 shell 从父级继承的文件创建掩码
# trap 设置的当前 traps
# 变量赋值语句或 set 设置的，或 shell 从父级继承的参数
# 执行期间定义的，或 shell 从父级继承的函数
# 调用时默认配置的、命令参数启用的，或 set 启用的选项
# shopt 启用的选项
# alias 定义的别名
# 包括后台任务在内的各进程 pid 值，以及 $$ 和 $PPID 的值


# 单独的执行环境
# shell 打开的文件，以及该命令的重定向所做的修改
# 当前工作目录
# 文件创建掩码
# export 导出的变量和函数，以及变量赋值等为命令导出的变量
# traps 被重置为 shell 从父级继承的值


# subshell 执行环境
# 为 shell 执行环境的副本
# subshell 执行环境中 traps 被重置为 shell 从父级继承的值
# --
( var=1 python3 <<<'import os; print(os.environ["var"])' ) # 1
( var=1; python3 <<<'import os; print(os.environ["var"])' ) # error:KeyError
( export var=1; python3 <<<'import os; print(os.environ["var"])' ) # 1
( a=1; ( a=2; echo $a ); echo $a ) # 2 1
```

环境变量

```bash
# 程序被调用时会得到一个称为 environment 的字符串序列，其为键值对序列，格式为 name=value
# 使用 export 和 declare 可以管理环境变量中的变量和函数
# 命令或函数的执行环境可以通过在其前面添加参数分配来临时扩充，这些赋值语句只影响该命令的执行环境
# 若设置 set -k 则命令行的所有参数分配都会添加到执行环境中，而不仅仅是命令名前的参数分配
# 调用外部命令时 $_ 会被设置为命令的完整路径并传递给命令的执行环境
# --
( a=1 bash -c 'echo $a' a=2; set -k; a=1 bash -c 'echo $a' a=2 ) # 1 2
python3 <<<'import os; print(os.environ["_"])' # /usr/bin/python3
```

退出状态

```bash
# 命令的退出状态为 0 则表示成功，非 0 则表示失败，其值在 0-255 之间
# 若命令在致命信号 N 上终止则 bash 使用 128+N 作为退出状态
# 若未找到命令则返回状态 127 若找到命令但无可执行权限则返回状态 126
# 内置命令使用退出状态 2 来表示无效选项、缺少参数等使用错误的情况
```

信号

```bash
# SIGINT 信号会被处理，因此 wait 可中断
# 收到 SIGINT 信号时会跳出任何正在执行的循环
# 由 bash 启动的非内置命令的信号处理会被设置为 shell 从其父级继承来的值
# shell 会在收到 SIGHUP 信号后退出，交互式 shell 退出之前会将该信号重新发送给所有运行或停止的作业
# 若无需 shell 向作业发送 SIGHUP 信号，可使用 disown 将其从作业表中删除，或使用 disown -h 标记为不接收 SIGHUP 信号
# 若设置 shopt -s huponexit 则交互式登录 shell 退出时会向所有作业发送 SIGHUP 信号
# 若 bash 在等待命令执行完成时收到信号而触发 traps 则该 traps 会在命令执行完成后再执行
# 若 bash 在使用 wait 等待异步命令完成时收到信号而触发 traps 则 wait 立即以大于 128 的状态返回并立刻执行 traps
```

脚本

```bash
# 文件作为 bash 第一个非选项参数且没有 -c 或 -s 选项，则创建非交互式 shell 读取并执行文件中的命令
# shell 在当前目录中查找该文件，若没有则在 $PATH 中查找
# 当 bash 运行脚本时 $0 会被设置为文件名，其余的参数（若有）则作为位置参数
# 可使用 chmod 使 shell 脚本具有可执行权限，以 ./script.sh 或 /path/script.sh 的类似命令的方式执行脚本
  # 脚本在重新初始化的 subshell 中执行，像新的 shell 一样，但在脚本不使用 #! 语法时父级 hash 的命令路径记录会保留
  # 若脚本第一行使用 #! 语法，则该行其余部分指定脚本的解释器及可选参数，因此可以指定 bash 或 awk 或 perl 等语言
  # 若使用 bash script.sh 的方式执行脚本则会忽略脚本的 #! 语法
  # 若使用 #!/usr/bin/env bash 语法，则使用 $PATH 中找到的第一个 bash 来解释脚本
# --
( cd /tmp; echo $'#!/usr/bin/env python3\nimport os\nprint(os.environ["_"])' >a.sh; chmod +x a.sh; ./a.sh ) # ./a.sh
```

协程

```bash
coproc [name] cmds [redirections]
# 命令在 subshell 异步执行
# --
coproc ls # [1] 876  [1]+  Done  coproc COPROC ls $LS_OPTIONS
```

并行

```bash
parallel [opts] [cmds [args]] ...
# 并行执行命令
# --
find /usr -maxdepth 1 -type d -name b* | parallel echo 'find: {}' # find: /usr/bin
```

命令的历史编号和命令编号

```bash
# 命令的历史编号是其在历史记录列表中的位置，其中可能包括从历史记录文件中恢复的命令
# 命令的命令编号是其在当前 shell 会话期间执行的命令序列中的位置
```

提示信息

```bash
# bash 在输出每个主要提示之前检查 PROMPT_COMMAND 数组（若有）的值，将每个元素视为命令并按顺序执行
# 变量 PS0 PS1 PS2 PS4 可使用特殊转义序列
\d # 日期，格式为 "Weekday Month Date" 例如 "Tue May 26"
\D{format} # 使用 strftime 中的格式字符 format 来输出日期时间字符串，空的 format 会使用特定于语言环境的时间格式
\h # 主机名，直至第一个 . 字符
\H # 主机名
\j # 当前由 shell 管理的作业数
\l # shell 的终端设备名的 basename
\n # 换行
\r # 回车
\s # shell 的名称，为 $0 的 basename 即最后一个斜杠之后的部分
\t # 时间，为 24 小时制 HH:MM:SS 格式
\T # 时间，为 12 小时制 HH:MM:SS 格式
\@ # 时间，为 12 小时制 HH:MM AM/PM 格式
\A # 时间，为 24 小时制 HH:MM 格式
\u # 当前用户的用户名
\v # bash 版本号，例如 5.1
\V # bash 版本补丁号，例如 5.1.4
\w # 当前工作目录，若为 $HOME 则缩写为波浪号，实际显示受 PROMPT_DIRTRIM 变量影响
\W # 为 $PWD 的 basename 若为 $HOME 则缩写为波浪号
\! # 这个命令的历史编号
\# # 这个命令的命令编号
\$ # 若 euid 是 0 则为 # 否则为 $
\nnn # ASCII 码为八进制值 nnn 的字符
\\ # 一个反斜杠
\[ # 开始非打印字符序列，可用于将终端控制序列嵌入到提示中
\] # 结束非打印字符序列
```

命令行编辑

```bash
# 使用 Readline 库实现，交互式 shell 默认启用，可使用 --noediting 选项禁用


# 组合键

Ctrl-b
# 向行首移动一个字符

Ctrl-f
# 向行尾移动一个字符

Ctrl-d
# 删除光标下的字符，若当前行为空行则会退出 shell

Ctrl-_
Ctrl-x Ctrl-u
# 撤消上一个编辑命令直至空行

Ctrl-a
# 移动到行首

Ctrl-e
# 移动到行尾

Meta-f
Alt-Shift-f
Ctrl-➡
# 向行尾移动一个单词

Meta-b
Alt-Shift-b
Ctrl-⬅
# 向行首移动一个单词

Ctrl-l
# 清除屏幕，在顶部重新打印当前行

Ctrl-k
# 删除当前光标到行尾之间的文本

Meta-d
Alt-Shift-d
# 删除当前光标到单词结尾之间的文本

Ctrl-w
# 删除当前光标到单词开头之间的文本

Ctrl-y
# 将最近删除的文本复制到当前光标

Ctrl-r
# 在历史记录中从新到旧搜索特定字符串

RET
# 中止搜索并执行历史列表中的命令

ESC
Ctrl-j
# 中止搜索且历史条目成为当前行

Ctrl-g
# 中止搜索并恢复当前行


# 初始化文件
# 变量 INPUTRC 指定的文件，或默认 ~/.inputrc 文件，或最终默认文件 /etc/inputrc
# 使用 Ctrl-x Ctrl-r 组合键重新加载初始化文件

set variable value
# 设置变量
enable-bracketed-paste # 括号粘贴模式，默认 on
enable-meta-key # 启用支持的 Meta 键，默认 on
# --
set editing-mode vi

Control-u: universal-argument
Meta-Rubout: backward-kill-word
Control-o: "> output"
# 组合键绑定

"\C-u": universal-argument
"\C-x\C-r": re-read-init-file
"\e[11~": "Function Key 1"
# 组合键绑定
\C- # Ctrl 前缀
\M- # Meta 前缀
\d # 删除键
# --
"\C-p": "echo 1"
"\C-p": clear-screen

$if [mode | term | version | application | variable] ... [$else ...] $endif
# 条件判断
# mode 模式，例如 $if mode=vi
# term 终端，例如 $if term=linux
# version 版本，例如 $if version >= 7.0
# application 应用，例如 $if Bash
# variable 变量，例如 $if editing-mode == emacs
# --
$if version >= 7.0
  set show-mode-in-prompt on
$endif

$include
# 读取指定配置文件
# --
$include /etc/inputrc


# 可用命令
# 可使用 bind 命令查看

beginning-of-line (C-a)
# 移动到当前行的开头

end-of-line (C-e)
# 移动到当前行的结尾

complete (TAB)
# bash 的补全
# 若以 $ 开头则视为变量名进行补全
# 若以 ~ 开头则视为用户名进行补全
# 若以 @ 开头则视为主机名进行补全
# 或视为命令（包括别名、函数）进行补全
# 若都没有匹配则视为文件名进行补全

re-read-init-file (C-x C-r)
# 重新加载 inputrc 文件


# 命令补全
# 在空行开头（空命令）进行补全时使用 complete -E 定义的补全规则
# 若命令带完整路径则优先使用带完整路径的补全规则，若没有则尝使用最后一个斜杠后面的命令的补全规则
# 若都没有找到对应的补全规则，则使用 complete -D 定义的默认补全规则
# 若没有默认补全规则，则尝试对命令使用别名扩展并查找扩展后的命令的补全规则
# 若找到了补全规则，则使用该规则生成匹配词列表，若没找到则使用 bash 的补全
# 按补全规则操作并且只返回前缀匹配的候选词，若文件或目录补全使用了 -f 或 -d 选项则 FIGNORE 变量用于候选词过滤
# 接下来使用 -G 定义的文件名扩展模式对应的补全规则生成候选词（不需要前缀匹配）并使用 FIGNORE 变量用于候选词过滤
# 接下来使用 -W 定义的字符串参数，使用 $IFS 对字符串进行拆分而后执行扩展，扩展后的结果进行前缀匹配并成为候选词
# 接下来调用 -F 或 -C 定义的函数或命令，且变量 COMP_LINE COMP_POINT COMP_KEY COMP_TYPE 被使用
  # 若调用的是函数则变量 COMP_WORDS COMP_CWORD 也被使用
  # 当函数或命令调用时 $1 是命令名 $2 是正在补全的单词 $3 是正在补全的单词之前的单词，完全由函数或命令控制过滤规则
# 首先调用 -F 定义的函数，该函数可使用 compgen compopt 在内的各种命令，而后必须将可能的补全放在 COMPREPLY 数组中
# 而后调用 -F 定义的命令，该命令的执行环境等同于命令替换的执行环境，而后必须将可能的补全按每行一个输出到标准输出
# 生成所有可能的补全后对结果列表执行 -X 指定的过滤规则，过滤规则是用于文件名扩展的 pattern
  # 在规则中的 & 字符会替换为正在补全的单词
  # 与规则匹配的补全都会被删除，若使用前导 ! 字符则与规则不匹配的补全都会被删除
  # 若设置了 shopt -s nocasematch 则匹配不区分大小写
# 最后将 -P 和 -S 指定的前缀和后缀添加到补全列表中的每个成员，并将结果作为可能的补全列表返回给 Readline 补全代码
# 若先前的操作未生成任何匹配项且定义补全规则时设置了 complete -o dirnames 则尝试进行目录名补全
# 若定义补全规则时设置了 complete -o plusdirs 则在生成的补全列表后添加目录名补全的匹配列表
# 若有补全规则，则使用规则生成的补全列表，不执行 bash 的补全，不执行 Readline 的文件名补全
# 若定义补全规则时设置了 complete -o bashdefault 且补全规则没有匹配项则会尝试 bash 的补全
# 若定义补全规则时设置了 complete -o default 且补全规则和 bash 的补全（若有）都没有匹配项则会尝试 Readline 的默认补全
# 使用目录补全时会依据 Readline 变量 mark-directories 的值决定是否在目录的符号链接后添加斜杠，即 / 字符
# 补全函数可以返回状态码 124 以重新执行补全，该操作将重新寻找对应的补全规则并进行补全，可用于实现动态加载补全规则


# 动态加载命令补全
# 设置默认的补全规则实现动态加载命令的补全规则
_completion_loader() { source "/etc/bash_completion.d/$1.sh" >/dev/null 2>&1 && return 124; }
complete -D -F _completion_loader -o bashdefault -o default


# 补全脚本
# 命令 abc 补全
# abc 有 opt1 opt2 两个参数
# opt1 有 opt1arg1 opt1arg2 两个参数
# opt2 有 opt2arg1 opt2arg2 两个参数
# 没有对应补全时使用 bash 和 Readline 的补全
_comp_abc(){
  local cur="$2"
  local pre="$3"

  local opts="opt1 opt2"
  local opt1="opt1arg1 opt1arg2"
  local opt2="opt2arg1 opt2arg2"

  if (( COMP_CWORD == 1 )); then
    COMPREPLY=( $(compgen -W "$opts" -- "$cur") )
    return
  fi

  case "$pre" in
    opt1)
      COMPREPLY=( $(compgen -W "$opt1" -- "$cur") )
      ;;
    opt2)
      COMPREPLY=( $(compgen -W "$opt2" -- "$cur") )
      ;;
  esac
}
complete -o bashdefault -o default -F _comp_abc abc
```

调用

```bash
# 交互式 shell
# 没有指定选项参数（或指定 -s 但没有 -c 选项）启动的 bash 且输入和错误输出都连接到终端，或使用 -i 选项启动的 bash
# 交互式 shell 通常从用户终端读取和输出
# -s 选项可在启动交互式 shell 时设置位置参数


# 判断是否为交互式 shell
# 变量 $- 在交互式 shell 中包含 i 字符，在非交互式 shell 中不含该字符
# 变量 $PS1 在交互式 shell 中被设置，在非交互式 shell 中为 unset
# --
case "$-" in *i*) echo Interactive;; *) echo NotInteractive;; esac # Interactive
if [[ -z "$PS1" ]]; then echo NotInteractive; else echo Interactive; fi # Interactive


# 受限制的 shell
# 以名称 rbash 启动，或调用时提供了 --restricted 或 -r 选项
# 受限制的 shell 不允许或不执行部分操作，这些限制在读取启动文件后执行
# 若命令是 shell 脚本，在因执行而生成的 shell 中这些限制会被关闭，可使用 #!/bin/bash -r 或 #!/bin/rbash 开启限制
# 现代系统提供了更安全的方法来实现受限环境，例如 jails zones containers


# POSIX 模式
# 使用 --posix 启动参数或 set -o posix 使得 bash 的行为符合 POSIX 模式
# 使用 sh 调用时 bash 在读取启动文件后进入 POSIX 模式


# 兼容模式
# 允许用户从以前的版本中选择与新版本不兼容的行为，目前同时只能有一个兼容级别，每个级别选项都是互斥的
# 应在 bash-5.0 及更高版本上使用 BASH_COMPAT 变量控制兼容模式


# 启动参数
bash [--opts] [opts] [args ...]
--posix # 修改 bash 的默认行为以符合 POSIX 标准
-c string # 从 string 中读取并执行命令，若之后有跟随参数则第一个设置为 $0 其余设置为位置参数
-i # 使 shell 以交互方式运行
# --
bash -c 'echo 1' # 1
bash -ci $'alias a=pwd\na' # /root


# bash 作为交互式登录 shell 调用或使用 --login 选项调用
# 读取并执行 /etc/profile 文件中的命令
# 按顺序查找 ~/.bash_profile 或 ~/.bash_login 或 ~/.profile 中第一个存在且可读的，读取并执行文件（若有）中的命令
# 可使用 --noprofile 禁止上述行为
# 退出时读取并执行 ~/.bash_logout 文件（若有）中的命令


# bash 作为交互式非登录 shell 调用
# 读取并执行 ~/.bashrc 文件（若有）中的命令
# 可使用 --norc 禁止上述行为
# 可使用 --rcfile file 指定读取 file 文件而不是读取 ~/.bashrc 文件
# 通常 ~/.bash_profile 文件内会包含 if [ -f ~/.bashrc ]; then . ~/.bashrc; fi 的配置加载代码


# bash 作为非交互式 shell 调用
# 扩展 BASH_ENV 变量（若有）的值，将其作为需要读取和执行的文件名，类似 if [ -n "$BASH_ENV" ]; then . "$BASH_ENV"; fi
# 变量 PATH 的值不会用于搜索变量 BASH_ENV 的值指定的文件名
# 若使用 --login 选项则会尝试从登录 shell 的启动文件中读取和执行命令


# bash 作为 sh 命令调用
# bash 会模仿 sh 的启动行为，同时符合 POSIX 标准
# bash 在读取启动文件后进入 POSIX 模式
# 作为交互式登录 shell 调用或使用 --login 选项调用
  # 按顺序读取并执行 /etc/profile 和 ~/.profile 文件（若有）中的命令
  # 可使用 --noprofile 禁止上述行为
# 作为交互式非登录 shell 调用
  # 扩展 ENV 变量（若有）的值，将其作为需要读取和执行的文件名
  # 不会尝试读取其他启动文件，因此 --rcfile 选项无效
# 非交互式 shell 调用
  # 不会尝试读取任何启动文件


# bash 在 POSIX 模式下调用
# 在 POSIX 模式下调用和使用 --posix 选项一样遵循 POSIX 标准
# 交互式 shell 调用会扩展 ENV 变量（若有）的值，将其作为需要读取和执行的文件名
# 不会尝试读取其他启动文件


# bash 由远程 shell 守护进程调用
# bash 判断其标准输入是否指向网络连接，就像由 rshd 或 sshd 的远程守护进程调用一样
# 读取并执行 ~/.bashrc 文件（若有）中的命令，若作为 sh 调用则不执行此操作


# bash 调用时 EUID/EGID 和 RUID/RGID 不相等
# 不读取启动文件
# 不继承环境中的 shell 函数
# 忽略环境中的 SHELLOPTS BASHOPTS CDPATH GLOBIGNORE 变量
# 若未指定 -p 选项则将 EUID/EGID 设置为 RUID/RGID 若指定了该选项则不重置 EUID/EGID
```

历史记录

```bash
# 历史记录变量
HISTSIZE
# 设置条目上限，默认 500

HISTFILE
# 设置文件位置，默认 ~/.bash_history

HISTTIMEFORMAT
# 设置条目日期格式

HISTCONTROL
HISTIGNORE
# 设置如何保存命令


# 历史扩展
# 默认使用 ! 字符引入历史扩展
# 在读取完整行后，在拆分单词前，在每行单独执行历史扩展


## 事件指示符
!
# 引入历史扩展，除非后面紧跟 空格 制表符 行尾 = (

!n
# 第 n 个历史命令

!-n
# 往后数第 n 个历史命令

!!
# 上一个命令，等同于 !-1

!str
# 最近的以 str 开头的历史命令

!?str[?]
# 最近的包含 str 的历史命令

^str1^str2^
# 重复上一个命令，将 str1 替换为 str2 等同于 !!:s^str1^str2^

!#
# 到目前为止输入的整个命令行


## 单词指示符
# 使用 : 字符分隔事件指示符与单词指示符，若单词指示符以 ^ $ * - % 开头则可以省略
# 单词从行首开始编号，第一个单词用 0 表示，单词被插入到当前行时，会由单个空格分隔

!!
# 上一个命令

!!:$
# 上一个命令的最后一个参数，可简写为 !$

!fi:2
# 最近的以 fi 开头的历史命令的第二个参数

0
# 第 0 个单词

n
# 第 n 个单词

^
# 第一个参数，即第一个单词

$
# 最后一个参数

%
# 最近的 ?str? 搜索匹配的第一个单词

x-y
# 一系列单词，其中 -y 为 0-y 简写

*
# 除了第 0 个外的所有单词，等同于 1-$

x*
# 为 x-$ 缩写

x-
# 类似 x* 但不含最后一个单词


## 修饰符
# 每个修饰符以 : 字符开头

h
# 删除路径名最后的尾部

t
# 只保留路径名最后的尾部

r
# 删除最后一个结尾后缀

e
# 只保留最后一个结尾后缀

p
# 输出新的命令但不执行

q
# 引用替换的单词以避免进一步的替换

x
# 类似 q 但是会进行分词

s/old/new/
# 把第一个 old 替换为 new
# 可以使用其他字符作为分隔符
# 若最后一个分隔符是输入行的最后一个字符则为可选的
# & 字符在 new 中会被替换为 old

&
# 重复前一次替换

g
a
# 修改替换行为，与 s 或 & 一同使用

G
# 对每个单词应用一次 s 或 & 的替换
```

内置命令

```bash
# 命令若接受以 - 开头的选项，则可使用 -- 表示选项的结束，即 -- 之后的内容都被视为参数而不是选项
# 命令 : true false test [ 不接受选项，也不对 -- 特殊处理
# 命令 exit logout return break continue let shift 可以接受以 - 开头的参数，无需使用 --


# 特殊内置命令
break : . continue eval exec exit export readonly return set shift trap unset
# bash 不在 POSIX 模式时与普通内置命令没有区别，在 POSIX 模式时有些许区别
  # 特殊命令的查找在 shell 函数之前
  # 特殊命令返回失败状态则非交互式 shell 直接退出
  # 特殊命令前的变量分配在命令完成后在 shell 执行环境依旧生效
# --
( set -o posix; a=1 :; echo ${a-unset} ) # 1
( set +o posix; a=1 :; echo ${a-unset} ) # unset


# 同 sh 的命令


: [args]
# 除了扩展参数和执行重定向之外什么都不做
# 返回状态为 0
# --
: && echo T # T
if [[ str ]]; then echo 1; else :; fi # 1


. filename [args]
source filename [args]
# 从 filename 中读取命令并在当前 shell 上下文执行
# 若 filename 不含斜杠则在 $PATH 中查找，若未找到且 bash 不在 POSIX 模式则还会在当前目录中查找
# --
( set -- a b; . <(echo 'echo $@') ) # a b
( set -- a b; source <(echo 'echo $@') ) # a b
( . <(echo 'echo $@') arg1 arg2 ) # arg1 arg2
( source <(echo 'echo $@') arg1 arg2 ) # arg1 arg2


break [n]
# 从循环中跳出
# 其中 n 为循环层级，其值应大等于 1
# --
( for i in {1..3}; do for j in {1..3};do echo "i:$i-j:$j"; break 2; done; done ) # i:1-j:1


cd [args] [directory]
# 切换当前工作目录
# 若省略 directory 则默认为 $HOME 目录
# 若 directory 为 - 则转换为 $OLDPWD 目录
# --
( cd /tmp; mkdir -p a1/b2; ln -sf a1/b2 c1; cd -P c1 && pwd; cd -; cd -L c1 && pwd ) # /tmp/a1/b2 /tmp /tmp/c1


continue [n]
# 开始循环的下次迭代
# 其中 n 为循环层级，其值应大等于 1
# --
( for i in {1..3}; do for j in {1..3};do echo "i:$i-j:$j"; continue 2; done; done ) # i:1-j:1 i:2-j:1 i:3-j:1


eval [args]
# 读取并执行 args 合并后形成的命令
# 退出状态为该命令的退出状态
# --
eval 'echo $BASHPID' # 1000
eval 'echo' '$BASHPID' # 1000


exec [args] [cmds [args]] [redirections]
# 若提供 cmds 则该命令直接替换当前 shell 进程而不是创建新进程执行
# 若不提供 cmds 则可以使用重定向来影响当前 shell 环境
# 非交互式 shell 中 cmds 无法执行则 shell 退出，若设置 shopt -s execfail 则变为返回失败状态
# 交互式 shell 中 cmds 无法执行则返回失败状态
# 在 subshell 中若 exec 失败则无条件退出
# --
( exec 33>/tmp/file; ll /dev/fd/33 ) # l-wx------. /dev/fd/33 -> /tmp/file
( exec -c printenv ) # 没有输出，没有环境变量，干净的执行环境


exit [n]
# 退出 shell 并将状态 n 返回给父级
# 若省略 n 则返回值为最后执行的命令的退出状态
# --
( trap 'echo before exit $?' EXIT; exit 3 ) # before exit 3


export [args] [name[=value]]
# 修改环境变量
# --
( a=str; bash -c 'echo "1.$a"'; export a; bash -c 'echo "2.$a"' ) # 1. 2.str


getopts optstring name [args]
# 用于脚本中解析位置参数
# 默认解析位置参数，若提供 args 则改为解析这些参数
# optstring 为要识别的选项字符，若选项字符后跟一个 : 则表示该选项有一个参数，其中 : 和 ? 不能用作选项字符
# 每次调用时 getopts 执行以下操作
  # 将 name 设置为下个选项，若该选项有参数则将 OPTARG 设置为对应的参数
  # 将 OPTIND 设置为下次调用时要处理的参数的索引
    # 在调用 shell 或脚本时 OPTIND 会被初始化为 1 但是 shell 不会自动重置 OPTIND
    # 在同一个 shell 多次调用 getopts 需要在多次调用的间隔手动重置 OPTIND
# 选项解析结束时 getopts 以大于 0 的返回值退出，而 OPTIND 设置为第一个非选项的参数的索引并且 name 设置为 ?
# 无效选项或缺少选项参数时默认输出错误，若 optstring 第一个字符为 : 则不输出错误，若 OPTERR 为 0 则始终不输出错误
# 无效选项时 name 设置为 ? 同时若不忽略错误则输出错误并取消 OPTARG 否则不输出错误并将 OPTARG 设置为选项字符
# 缺少选项参数时若不忽略错误则输出错误且 name 设置为 ? 并取消 OPTARG 否则 name 设置为 : 且 OPTARG 设置为选项字符
# --
( set -- -a a1 -b b1 -c
  while getopts ':a:b:' name
    do case $name in
      a)
        echo "a@$OPTARG@$OPTIND";;
      b)
        echo "b@$OPTARG@$OPTIND";;
      ?)
        echo "?@$OPTARG@$OPTIND";;
    esac
  done ) # a@a1@3 b@b1@5 ?@c@6
( while getopts ':a:b:' name -a a1 -b b1 -c
    do case $name in
      a)
        echo "a@$OPTARG@$OPTIND";;
      b)
        echo "b@$OPTARG@$OPTIND";;
      ?)
        echo "?@$OPTARG@$OPTIND";;
    esac
  done ) # a@a1@3 b@b1@5 ?@c@6


hash [args] [name]
# 管理记录命令的完整路径，用以减少对 $PATH 的搜索
# --
( ls >/dev/null; hash | grep ls ) # 1 /usr/bin/ls
( hash -p /xxx/ls ls; hash | grep ls ) # 0 /xxx/ls


pwd [args]
# 输出当前工作目录的绝对路径
# --
( cd /tmp; mkdir -p a/b; ln -sf a/b c; cd c; pwd -L; pwd -P ) # /tmp/c /tmp/a/b


readonly [args] [name[=value]] ...
# 将每个 name 标记为只读
# --
( fn(){ :; }; readonly -f fn; fn(){ :; } ) # error:readonly
( readonly -a a=(1 2 3); echo ${a[@]}; a=1 ) # 1 2 3 error:readonly
( readonly -A a=([a]=1 [b]=2 [c]=3); echo ${a[@]@K}; a=1 ) # c "3" b "2" a "1" error:readonly


return [n]
# 停止函数的执行并返回值 n 给调用者
# 也可用于停止 source 的脚本执行
# 若省略 n 则返回状态为最后一个命令的退出状态
# 若提供 n 则返回值为 n 最后 8-bit 的数值，即返回值应在 0-255 之间
# --
( fn(){ return 12; }; fn; echo $? ) # 12
( fn(){ return 280; }; fn; echo $? ) # 24


shift [n]
# 位置参数列表向左移动 n 位
# 位置参数 n+1 ... $# 变为 $1 ... $#-n 原位置参数 n+1 ... $# 被 unset
# n 取值 $# >= n >= 0 之间，若 n 等于零或大于 $# 则位置参数列表不变，若省略 n 则默认值为 1
# --
( set -- a b c; echo "$@ ."; shift; echo $@ ) # a b c . b c


[ condition ]
test condition
# 计算条件表达式
# 若为真则返回状态为 0 若为假则返回状态为 1 若表达式错误则返回状态为 2
# --
test ab = ab; echo $? # 0
[ ac = ab ]; echo $? # 1


times
# 输出 shell 及其子进程使用的用户和系统时间


trap [args] [cmds] [sigspec ...]
# shell 在收到信号 sigspec 时读取并执行 cmds 中的命令，其中 sigspec 为信号名或对应编号
# 若省略 cmds 且只有一个 sigspec 则该信号处理被重置，若 cmds 为 - 则所有 sigspec 信号处理都被重置
# 若 cmds 是空字符串则所有 sigspec 信号都会被 shell 及其调用的命令忽略
  # 调用 shell 时忽略的信号不能被捕获或重置，未被忽略的信号处理会在 subshell 中被重置
  # 在 bash 的 subshell 中可以使用 trap 看到父级的信号处理但实际上不会生效
# 信号名称不区分大小写且 SIG 前缀是可选的
# --
trap 'echo hello' EXIT; ( trap; exit ); trap - EXIT # trap -- 'echo hello' EXIT
( trap 'echo hello' EXIT; trap; exit ) # trap -- 'echo hello' EXIT  hello


umask [args] [mode]
# 设置 shell 的文件创建掩码
# 若 mode 以数字开头则将其视为八进制数，否则视为类似 chmod 的符号模式掩码
# 视为八进制数时权限计算会使用 7 减去对应数字，即 022 对应 755 的权限
  # 默认不给普通文件执行权限，因此目录使用 7 减、普通文件使用 6 减，即 022 为目录 755 权限、为普通文件 644 权限
# --
( cd /tmp; umask; touch a_file; ll a_file; mkdir a_dir; ll -d a_dir ) # 0022  -rw-r--r-- a_file  drwxr-xr-x a_dir


unset [args] [name]
# 删除 name 变量或函数
# 若指定的变量不存在则删除同名的函数（若有）
# 只读变量和只读函数不能被 unset
# 部分 shell 变量被 unset 会失去其特殊行为
# --
( fn=1; fn(){ :; }; echo $fn; type fn; unset fn; echo ${fn-unset}; type fn ) # 1 function unset function
( fn(){ :; }; type fn; unset fn; type fn ) # function error:notfound
( a=1; declare -n ref=a; unset ref; echo ${a-unset} ) # unset
( a=1; declare -n ref=a; unset -n ref; echo ${a-unset} ) # 1


# bash 特有或扩展的命令


alias [args] [name[=value] ...]
# 使用 name 作为 value 的别名
# bash 执行输入命令前至少读取一行完整输入，别名在读取时展开而非执行时，因此同一行命令不受影响，别名在下一行生效
# --
( alias a='ls -l'; alias a ) # alias a='ls -l'


bind [args]
# 显示 Readline 绑定、设置 Readline 绑定、设置 Readline 变量
# --
( bind 'set enable-bracketed-paste on' ) # 启用括号粘贴模式
  # 括号粘贴模式 (bracketed paste mode) 下复制到终端的内容前后会添加特殊序列，避免因复制内容含有回车使得命令被执行
( bind -x '"\C-o":echo 1'; bind -X ) # "\C-o": "echo 1" 输入 Ctrl 和 o 则执行命令 echo 1
( bind '"\eh":forward-char'; bind -q forward-char ) # invoked via "\eh" 输入 Esc 和 h 则光标向右移动一个字符


builtin [builtin_cmds [args]]
# 将 args 作为命令参数来执行 shell 内置命令 builtin_cmds 并返回命令的退出状态
# 创建与内置命令同名的函数时可用于在同名函数内使用同名的内置命令
# --
( echo(){ builtin echo "args: $@ ."; builtin echo "$@"; }; echo -e '\0176' ) # args: -e \0176 . ~


caller [frame]
# 返回任何活动的子程序调用的上下文
# 若指定 frame 则显示当前执行调用堆栈中与该位置对应的行号、子程序名、源文件名
# --
bash <<<'out(){ n=0; while caller $n; do ((++n));done;}
  f1(){ out; }
  f2(){ f1; }
  f3(){ f2; }
  f3' # 2 f1 main  3 f2 main  4 f3 main


command [args] cmds [args ...]
# 仅执行 shell 内置命令或搜索 $PATH 找到的 cmds 命令，忽略同名的 shell 函数
# --
( function pwd(){ echo 1; }; command pwd; pwd ) # /root 1


declare [args] [name[=value] ...]
# 定义变量及属性
# 使用 - 设置对应属性，使用 + 关闭对应属性
# 在函数中使用则变量成为本地变量，可使用 -g 设置全局变量
-g # 在全局范围创建或修改变量
-a # 创建索引数组
-A # 创建关联数组
-i # 视为整数变量，赋值时执行算术表达式
-l # 赋值时大写转小写
-n # 设置 nameref 使变量指向另一个变量
-r # 创建只读变量，不能修改、不能取消只读、不能被 unset
-u # 赋值时小写转大写
-x # 将变量导出为环境变量，类似 export
# --
( declare -A arr=([a]=e1 [b]=e2); echo ${arr[a]} ) # e1
( declare -n ref=var; var=1; echo $ref ) # 1
( typeset -A arr=([a]=e1 [b]=e2); echo ${arr[a]} ) # e1
( typeset -n ref=var; var=1; echo $ref ) # 1


echo [args] [value ...]
# 以空格分隔和换行符结束的格式输出 value
# 不支持 -- 作为选项结束的语法
-n # 不输出结尾的换行
-e # 部分转义序列有特殊含义
  \b # 退格
  \n # 换行
  \t # 横向 tab
  \0[N] # 对应八进制值的 8-bit 字符，其中 [N] 为 0-3 个八进制数
  \x[N] # 对应十六进制值的 8-bit 字符，其中 [N] 为 1-2 个十六进制数
  \u[N] # 对应十六进制值的 unicode 字符，其中 [N] 为 1-4 个十六进制数
  \U[N] # 对应十六进制值的 unicode 字符，其中 [N] 为 1-8 个十六进制数
# --
echo -e '\0176 \x7E \u7E \U7E' # ~ ~ ~ ~
echo -e '\0346\0210\0221 \xE6\x88\x91 \u6211 \U6211' # 我 我 我 我
echo -e -- -e # -- -e


enable [args] [name ...]
# 启用和禁用内置命令，用以在不指定完整路径的情况下执行与内置命令同名的磁盘命令
# --
( enable -n echo; echo 1; hash -t echo ) # 1 /usr/bin/echo


help [args] [pattern]
# 显示有关内置命令的帮助信息
# --
help -d pwd # pwd - Print the name of the current working directory.


let expression [expression ...]
# 计算算术表达式
# 表达式的值为非 0 则返回状态为 0 否则返回状态为 1
# --
( a=3; let "a=4"; echo $a $? ) # 4 0
( a=3; let "a==4"; echo $a $? ) # 3 1
( a=3; let "a==3"; echo $a $? ) # 3 0
let 3 0; echo $? # 1
let 0 3; echo $? # 0


local [args] name[=value] ...
# 创建并赋值局部变量
# 仅在函数内部使用，使变量的可见范围仅限于该函数及其子函数
# 若 name 为 - 则使得 set 修改的选项仅在函数内生效
# --
( f(){ a=1; echo $a; }; f; echo $a ) # 1 1
( f(){ local a=1; echo $a; }; f; echo ${a-unset} ) # 1 unset
( f(){ set -e; }; f; err; err ) # err:notfound
( f(){ local -; set -e; }; f; err; err ) # err:notfound err:notfound


logout [n]
# 退出 shell 并将状态 n 返回给父级
# --
trap 'echo before logout $?' EXIT; logout 3 # before logout 3


mapfile [args] [array]
# 从标准输入或文件描述符读取行并赋值给索引数组 array 或变量 MAPFILE
-t # 从读取的每一行中删除尾随分隔符（默认为换行符）
# --
( mapfile -t arr <<<$'aa\nbb\ncc'; echo ${arr[@]@K} ) # 0 "aa" 1 "bb" 2 "cc"
( readarray -t arr <<<$'aa\nbb\ncc'; echo ${arr[@]@K} ) # 0 "aa" 1 "bb" 2 "cc"


printf [args] format [value]
# 使用 format 格式化输出 value
-v var # 不直接输出而是赋值给变量 var
format # 格式化字符串
  %b # 扩展转义序列，类似 echo -e
  %q # 以可直接用于输入的格式输出
  %(datefmt)T # 使用 strftime 中的格式字符 datefmt 来输出日期时间字符串
# --
printf '%d %f\n' 1 1.23 # 1 1.230000
printf '%b\n' '\0176' # ~
printf '%q\n' '$@' # \$@
printf '%(%F %T %Z)T\n' 0 # 1970-01-01 08:00:00 CST  其中 -1 为当前时间 -2 为调用 shell 的时间


read [args] [name ...]
# 从标准输入或文件描述符读取行并赋值给变量
# 变量比输入的值多则剩余的变量为空值，变量比输入的值少则最后一个变量获得剩余的值
# 若不提供 name 则输入保存至变量 REPLY
-a aname # 输入被赋值给索引数组 aname
-p prompt # 输出提示字符串 prompt
# --
( read <<<'abc'; echo $REPLY ) # abc
( read -a arr <<<'a b c'; echo ${arr[@]@K} ) # 0 "a" 1 "b" 2 "c"
( read a b c <<<'1 2'; echo $a . $b . ${c@Q} ) # 1 . 2 . ''
( read a b c <<<'1 2 3 4'; echo $a . $b . $c ) # 1 . 2 . 3 4


readarray
# 同命令 mapfile


source
# 同命令 .


type [args] [name ...]
# 表明如果将 name 用作命令时将如何解释
-t # 返回值为 alias function builtin file keyword 之一，若 name 不存在则不输出并以失败状态退出
# --
type -t notfound; echo $?; type -t echo; echo $? # 1 builtin 0


typeset
# 同命令 declare


ulimit [args] [limit]
# 对 shell 启动的进程进行资源控制
# --
( su - test -c 'ulimit -SH -u 1; ls | cat' ) # error:resource temporarily unavailable


unalias [args] [name ...]
# 取消 alias 设置的别名
# bash 执行输入命令前至少读取一行完整输入，别名在读取时展开而非执行时，因此同一行命令不受影响，别名在下一行失效
# --
( alias a='ls -l'; alias a; unalias a; type a ) # alias a='ls -l' error:notfound


# 未明确是否特有或扩展的命令


bg [jobspec ...]
# 在后台恢复每个暂停的作业


compgen [args] [word]
# 为 word 生成可能的补全并输出到标准输出


complete [args] name [name ...]
# 设置如何进行 name 的补全
-r [name] # 移除指定的或全部补全规则
-D # 设置默认的补全规则
-E # 空行开头（空命令）的补全规则
-F func # 当函数调用时 $1 是命令名 $2 是正在补全的单词 $3 是正在补全的单词之前的单词
  # 变量 COMP_LINE COMP_POINT COMP_KEY COMP_TYPE COMP_WORDS COMP_CWORD 被使用
-W wordlist # 可能的候选并从中选取匹配的成员


compopt [args] [name]
# 修改或显示指定的补全规则


dirs [-clpv] [+N | -N]
# 显示当前记录的目录列表
# 使用 pushd 命令将目录添加到列表中，使用 popd 命令从列表中删除目录
# 当前目录始终是堆栈中的第一个目录


disown [args] [jobspec ... | pid ... ]
# 从作业表中删除指定的作业或进程


fc [args] [cmd]
# 显示或编辑并重新执行历史命令


fg [jobspec]
# 在前台恢复作业 jobspec 并使其成为当前作业


history [args]
# 显示带有行号的历史列表
-c # 清理历史记录，配合 -w 可以清理记录文件
-w # 将当前历史记录写入文件


jobs [args] [jobspec]
# 显示当前的作业


kill [args] jobspec or pid
# 将 sigspec 或 signum 指定的信号发送给 jobspec 作业或 pid 进程
# 作业控制未启用时不接受 jobspec 参数，必须使用 pid


popd [-n] [+N | -N]
# 没有给出参数时 popd 从堆栈中删除顶层目录并 cd 到新的顶层目录
# 元素从 0 开始编号，从 dirs 列出的第一个目录开始，即 popd 等价于 popd +0


pushd [-n] [+N | -N | dir]
# 将当前目录保存在目录堆栈的顶部，然后 cd 到 dir 目录，此时 dir 会成为目录堆栈的顶部
# 没有给出参数时 pushd 会交换前两个目录并 cd 到对应目录


set [args] [value ...]
# 修改 shell 选项、设置位置参数、查看 shell 变量
# 使用 - 启用指定选项，使用 + 关闭指定选项，使用变量 $- 获取当前选项设置
-a # 修改或创建的变量或函数都视为环境变量
-e # 若命令退出状态非 0 则立即退出 shell
-n # 读取命令但不执行，可用于检查语法错误，对交互式 shell 不生效
-o optname # 设置对应选项
  pipefail # 设置管道的退出状态为最后一个非 0 退出状态或全部成功为 0
  posix # 修改 bash 的默认行为以符合 POSIX 标准
-x # 跟踪命令执行并输出信息，在输出的命令和参数前添加 $PS4 扩展后的值
-- # 若该选项没有跟随参数则 unset 位置参数，若有跟随参数则将其设置为位置参数
- # 选项结束标识，同时 -x 和 -v 选项会被关闭，并且若有剩余的参数则设置为位置参数，若没有则不修改位置参数
# --
( echo $-; set +m; echo $- ) # himBHs hiBHs
( set -a; a=1; bash -c 'echo $a' ) # 1
( set -e; err; echo 1 ) # error:notfound
bash -c 'echo 1; set -n; echo 2' # 1
( set -o pipefail; exit 1 | exit 2 | exit 0; echo $? ) # 2
( set -- a b c.; echo $@; set -- ; echo ${@-unset} ) # a b c. unset
( set -xv; set - a b c.; echo $@; set -; echo $@) # + set - a b c. a b c. a b c.


shopt [args] [optname ...]
# 切换 shell 的行为设置
-s optname # 启用对应选项
-u optname # 禁用对应选项
  autocd # 作为路径的命令会被视为 cd 的参数进而切换目录，仅交互式 shell 有效
  expand_aliases # 启用别名扩展，交互式 shell 默认启用
-o optname # 设置对应选项，同 set 命令的 -o optname 选项
  pipefail # 设置管道的退出状态为最后一个非 0 退出状态或全部成功为 0
# --
shopt -s autocd; /tmp; pwd; shopt -u autocd; cd # cd -- /tmp  /tmp
bash -c $'shopt -s expand_aliases\nalias a="pwd"\na' # /root
( shopt -s -o pipefail; set -o | grep pipefail ) # pipefail on


suspend [-f]
# 暂停 shell 的执行直至收到了 SIGCONT 信号


wait [args] [jobspec or pid ...]
# 等待每个 pid 进程或 jobspec 作业的子进程退出，并返回最后等待的命令的退出状态
# 若没有参数则等待所有正在运行的后台作业和最后执行的进程替换
# 作业控制未启用时不接受 jobspec 参数，必须使用 pid
```
<!-- endregion -->
