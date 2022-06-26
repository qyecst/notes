# 笔记

---
`bash`相关

```bash
# 安装
apt/yum install bash
https://www.gnu.org/software/bash
```

```bash
# 注释
# 以`#`开始的内容为注释
```

```bash
# 引用
" # 双引号中"$\`!*@"之类的符号有特殊含义
  # 双引号之间可以使用反斜杠转义双引号

' # 单引号中的内容一般原样输出
  # 单引号之间不能有单引号，使用反斜杠也不行

\ # 反斜杠为转义符用于保留下个字符的字面值
  # 若反斜杠在行尾且没被括起则视为续行
  # 可以对空格进行转义，被转义的空格不会引起分词等操作
    # 例
    ( set -- a1 b\ 2 c 3; echo $# . ${@@Q} ) # 4 . 'a1' 'b 2' 'c' '3'

$'' # 扩展为字符串并替换特定转义字符
  \b # 退格
  \n # 换行
  \r # 回车
  \t # 横向tab
  \v # 竖向tab
  \\ # 反斜杠
  \' # 单引号
  \" # 双引号
  \[N]  # 对应8进制值的ascii字符，[N]为1-3个8进制数
  \x[N] # 对应16进制值的ascii字符，[N]为1-2个16进制数
  \u[N] # 对应16进制值的unicode字符，[N]为1-4个16进制数
  \U[N] # 对应16进制值的unicode字符，[N]为1-8个16进制数
    # 例
    ( a=$'\176 \x7E \u7E \U7E'; echo "$a" ) # ~ ~ ~ ~
```

```bash
# 变量
name=expr # 直接定义变量并赋值，等号左右不能有空格

declare
typeset # 使用内建命令定义变量#TODO

$name
${name} # 获取变量的值
  # 使用`${name}str`可以避免后缀str被视为变量名的一部分
    # 例
    ( var=1; echo $var . "$varstr" . ${var}str ) # 1 . "" . 1str
```

```bash
# 特殊变量
$@
$* # 获取所有位置参数的值
  # 使用`$@`时，若无引号括起则为默认分词，若有引号括起则格式为"arg1" "arg2" "arg3"
  # 使用`$*`时，若无引号括起则为默认分词，若有引号括起则格式为"arg1${IFS}arg2${IFS}arg3"#TODO
    # 例
    ( set -- a b c; IFS=@; echo $* . "$*" . $@ . "$@" ) # 'a' 'b' 'c' . 'a@b@c' . 'a' 'b' 'c' . 'a' 'b' 'c'

$# # 获取位置参数的总数#TODO

${#N} # 第N个位置参数其值的长度

$? # 上一个命令的退出状态，0为正常退出

$$ # 当前shell的pid
  # 子shell中pid与父shell一样，若需要子shell的pid应使用`$BASHPID`#TODO

$0 # shell或脚本的名称
    # 例
    echo $0 # -bash
    bash -c 'echo $0' # bash
    /usr/bin/bash -c 'echo $0' # /usr/bin/bash

$N
${N} # 第N(N>=1)个位置参数的值
  # 当N为多位数时需要使用`${N}`的形式进行引用
```

```bash
# shell变量
$BASHPID # 当前shell的pid
  # 与`$$`不同在于`$$`在子shell中不会被重新初始化而是继承父shell的值

$BASH_COMMAND # 当前或即将执行的命令
  # 若是因`trap`而执行则显示为触发时正在执行的命令#TODO

$BASH_SUBSHELL # 初始值为0，每当进入子shell环境时增加1

$BASH_SOURCE # 数组，类似文件调用堆栈
  # 其中`${BASH_SOURCE[0]}`为当前脚本名称
  # 其中`${FUNCNAME[i]}`定义于`${BASH_SOURCE[i]}`且被`${BASH_SOURCE[i+1]}`调用

$FUNCNAME # 数组，类似函数调用堆栈，变量仅在函数执行时存在
  # 其中`${FUNCNAME[0]}`为当前执行的函数名，`${FUNCNAME[-1]}`为main

$PPID # 父进程的pid

$RANDOM # 返回一个0-32767的随机数
  # 对其进行赋值可以指定seed，即可以重复生成相同的随机数

$SRANDOM # 返回一个32bit的伪随机数
  # 不能指定seed，赋值没有效果

$FUNCNEST # 若值大于0则视为函数最大嵌套等级，超过限制的函数调用会导致整个命令的终止
  # 默认没有限制

$IFS # 内部字段分隔符，用于扩展后的分词、内建读取命令将行拆分成单词
  # 默认值为`$' \t\n'`即<space><tab><newline>#TODO

$PATH # 命令寻找路径，以冒号分隔的目录列表
  # 零长度目录表示当前目录，可能显示为<path>::<path>或<path>:或:<path>

$PS0 # 在读取命令之后且执行命令之前，由shell扩展并显示

$PS1 # 主要提示字符串，为shell左侧的提示字符，类似"user@host:~$ "#TODO

$PS2 # 辅助提示字符串，类似续行或多行输入的提示符"> "

$PS3 # 用于`select`的输入提示字符串#TODO

$PS4 # 在执行跟踪期间显示在命令之前，第一个字符可被复制多次以体现间隔级别，类似`set -x`后输出调试信息前的加号

$TMOUT # 若值大于0则视为读取的默认超时
  # 命令等待终端输入超时未到达则终止、交互式shell等待完整输入行超时未到达则退出
```

```bash
# 管道
| # 将前一个命令的标准输出作为后一个命令的标准输入
|& # 为`2>&1`的管道简写，将前一个命令的标准输出和标准错误都作为后一个命令的标准输入
  # 管道的返回状态为最后一个命令的退出状态，若设置了`pipefail`则返回状态为最后一个非0退出状态，或全部成功返回0
  # 管道中的每个命令都作为单独的进程在子shell中执行
    # 例
    echo 0:$$; echo 1:$BASHPID >&2 | echo 2:$BASHPID >&2 # 0:3683 2:4130 1:4129
    ( set -o pipefail; exit 1 | exit 3 | true; echo $? ) # 3
```

```bash
# 命令列表
cmd1 ; cmd2 # cmd1执行完成(不论执行成功或失败)，然后执行cmd2
cmd1 & cmd2 # 可视作一个后台一个前台同时执行cmd1和cmd2
cmd1 && cmd2 # cmd1执行成功后才会执行cmd2
cmd1 || cmd2 # cmd1执行失败后才会执行cmd2
    # 例
    time ( { sleep 3 && echo 1; } & { sleep 4 && echo 2; } ) # 3秒后输出1，4秒后输出2，总耗时4秒
```

```bash
# 命令分组
(cmd) # 在子shell中执行
{ cmd; } # 在当前shell中执行，命令以分号或换行结束，括号左右需要空格或分号等字符进行分词
    # 例
    echo $$; (echo $BASH_SUBSHELL; echo $$; echo $BASHPID) # 2853 1 2853 2896 其中`$$`继承父shell的值
    echo $$; { echo $BASH_SUBSHELL; echo $$; echo $BASHPID; } # 2853 0 2853 2853
```

```bash
# 表达式 #TODO
((expr)) # 计算算术表达式#TODO
[[ cond ]] # 计算条件表达式
    # 例
    [[ "some str" =~ ^[[:alnum:][:punct:][:blank:]]+$ ]]; echo $? # 0
```

```bash
# 判断/if语句
if [[ cond ]]; then
  cmd
fi
## ---
if (( expr )); then
  cmd1
else
  cmd2
fi
## ---
if [ cond ]; then
  cmd1
elif [[ cond ]]; then
  cmd2
elif (( expr )); then
  cmd3
else
  cmd4
fi
## ---
if [[ cond ]]; then cmd; fi
if (( expr )); then cmd1; else cmd2; fi
if [ cond ]; then cmd1; elif [[ cond ]]; then cmd2; elif ((expr)); then cmd3; else cmd4; fi
    # 例
    if [ "" ]; then echo 1; elif [[ "" ]]; then echo 2; elif ((0)); then echo 3; else echo 4; fi # 4
```

```bash
# 分支/select语句
# `break [n]`跳出n层，默认跳出当前
select name in expr; do
  cmd
  break
done
## ---
select name in expr; do cmd; break; done
    # 例
    ( select i in `ls`; do echo $i; break; done ) # 1) anaconda-ks.cfg 2) file

# 分支/case语句
# 使用`;;`区分执行代码块，使用`*`作为默认匹配
case name in
  expr1)
    cmd1
  ;;
  expr2)
    cmd2
  ;;
  *)
    cmd3
  ;;
esac
## ---
case name in expr1) cmd1;; expr2) cmd2;; *) cmd3;; esac
    # 例
    ( a=1; case $a in 0|1|2|3) echo num;; *) echo other; esac ) # num
```

```bash
# 循环/for语句
# `continue`直接开始下个循环，`break [n]`跳出n层，默认跳出当前
for name in expr; do
  cmd
done
## ---
for ((expr1; expr2; expr3)); do
  cmd
done
## ---
for name in expr; do cmd; done
for ((expr1; expr2; expr3)); do cmd; done
    # 例
    ( for i in `ls`; do echo $i.L; done ) # "anaconda-ks.cfg.L" "file.L"
    ( for i in {1..6..2}; do echo $i; done ) # 1 3 5
    for((;;)); do echo 1; sleep 1; done # 一直输出1
    ( for((i=0;i<3;++i)); do echo $i; done ) # 0 1 2

# 循环/while语句
# `continue`直接开始下个循环，`break [n]`跳出n层，默认跳出当前
while ((expr)); do
  cmd
done
## ---
while ((expr)); do cmd; done
    # 例
    ( a=3; while((a>0)); do echo $a; ((--a)); done ) # 3 2 1

# 循环/until语句
# `continue`直接开始下个循环，`break [n]`跳出n层，默认跳出当前
until ((expr)); do
  cmd
done
## ---
until ((expr)); do cmd; done
    # 例
    ( a=3; until((a<1)); do echo $a; ((--a)); done ) # 3 2 1
```

```bash
# 函数#TODO
name() {
  cmd
}
## ---
function name {
  cmd
}
## ---
function name() {
  cmd
}
## ---
name() { cmd; }
function name { cmd; }
function name() { cmd; }
    # 例
    ( function fn() { pwd; } >&2; fn >/dev/null ) # /root
```

```bash
# 数组/引用
${arr[idx]} # 获取数组arr中索引为idx的元素值
  # idx前往后数从0起，后往前数从-1起，即0为第一个元素，-1为最后一个元素

${arr} # 等价`${arr[0]}`获取第一个元素值

${arr[@]}
${arr[*]} # 获取数组内的所有元素值
  # 两者区别参考`$@`与`$*`的区别#TODO

${#arr[@]}
${#arr[*]} # 获取数组自身的长度

${#arr[idx]} # 获取数组索引为idx的元素值长度

${!arr[@]}
${!arr[*]} # 获取数组所有元素的索引名
  # 两者区别参考`$@`与`$*`的区别#TODO

arr+=(e1) # 索引数组，将元素e1添加至现有最大idx+1下标的位置

arr+=([k]=e1) # 关联数组，将元素e1添加至k下标位置
    # 例
    ( arr=(e1 e2 e3); echo ${arr[0]} ${arr[-1]} ) # e1 e3
    ( declare -A arr=([a]=e1 [b]=e2); IFS=@; echo "${!arr[@]}" . "${!arr[*]}" ) # "a" "b" . "a@b"

# 数组/创建
declare -a arr
local -a arr
readonly -a arr 使用内建命令的`-a`参数创建索引数组

arr=(e1 e2)
arr[idx]=e1 # 直接使用对应语法创建索引数组
    # 例
    ( a=(e1 e2); b[2]=e3; echo ${!a[@]} . ${a[@]} . ${!b[@]} . ${b[@]} ) # 0 1 . e1 e2 . 2 . e3

# 关联数组，使用内建命令的`-A`参数创建
declare -A arr
local -A arr
readonly -A arr
    # 例
    ( declare -A a=([k1]=e1 [k2]=e2); echo ${!a[@]} . ${a[@]} ) # k1 k2 . e1 e2
```

```bash
# 扩展/花括号扩展
{a,b,c} # 扩展为a b c三个元素
{a..b..step} # 在a到b间按步进step进行扩展；step可省略
    # 例
    echo /usr/{bin,lib,src} # /usr/bin /usr/lib /usr/src
    echo {1..3} . {1..6..2} . {6..1..-2} . {a..c} # 1 2 3 . 1 3 5 . 6 4 2 . a b c

# 扩展/波浪号扩展
~[user] # user用户的家目录，若不指定user则为当前用户家目录；user不存在则原样输出
    # 例
    echo ~ ~ftp ~sshd ~null # /root /var/ftp /var/empty/sshd ~null

# 扩展/参数扩展/值获取
$name
${name} # 获取变量name的值
  # 花括号是可选的，用于避免变量名受其后字符的影响，确定变量名边界

${!name} # 开头是`!`且name不是nameref引用则为间接扩展，即将name的值作为变量名，而后扩展该变量名
  # 若name为nameref引用则不执行完整的间接扩展，而是将其扩展为引用对象的名称
    # 例
    ( nameA=valueA; declare -n namerefB=nameA; nameC=nameA; echo ${!nameC} ${!namerefB} ) # valueA nameA

# 扩展/参数扩展/默认值
${name:-expr} # name为空或未设置，返回expr，name不变
${name:=expr} # name为空或未设置，返回expr，name赋值expr
${name:?expr} # name为空或未设置，报错，报错信息expr
${name:+expr} # name为空或未设置，返回空，否则返回expr，name不变
    # 例
    ( a=''; echo ${a:-null} . "$a" . ${a:=$(pwd)} $a ) # null . "" . /root /root
    ( a=''; echo ${a:?invalid} ) # -bash: a: invalid
    ( a=''; echo "${a:+notice}" . ${a:=$(pwd)} ${a:+string} ) # "" . /root string

# 扩展/参数扩展/变量名
${!prefix@}
${!prefix*} # 获取以prefix开头的变量名
  # 两者区别参考`$@`与`$*`的区别#TODO

${!name[@]}
${!name[*]} # 获取数组name所有元素的索引名
  # 两者区别参考`$@`与`$*`的区别#TODO

${#name} # 获取变量name值的长度
  # 若name为`@`或`*`时参考位置参数的总数#TODO
  # 若name为数字时参考位置参数其值的长度#TODO
  # 若name为数组且下标为`[@]`或`[*]`时参考数组长度#TODO
  # 若name为数组且下标为数字时参考数组元素其值的长度#TODO
    # 例
    ( a_1=1; a_2=2; a_3=3; IFS=@; echo "${!a_*}" . "${!a_@}" ) # a_1@a_2@a_3 . a_1 a_2 a_3

# 扩展/参数扩展/截取
${name:offset:length} # 从offset起截取length长度作为返回值，name不变
  # offset左往右从0计，右往左从-1计
  # length正数为长度，负数为距离末尾的偏移量
  # offset负数时需要添加空格`${name: -offset:length}`以避免与`${name:-expr}`混淆#TODO
  # 若name为`@`或`*`时截取位置参数列表，offset同上length须大于0，两者区别参考`$@`与`$*`的区别#TODO
  # 若name为数组且下标为`[@]`或`[*]`时截取数组元素列表，offset同上length须大于0，两者区别参考`$@`与`$*`的区别#TODO

${name#expr}
${name##expr} # 删除name值中符合匹配的前缀，其必须从前缀开始就匹配
  # `#`最短匹配，`##`最长匹配
  # 返回截取后的结果，name不变
  # 若name为`@`或`*`时操作将依次应用于每个位置参数
  # 若name为数组且下标为`[@]`或`[*]`时操作将依次应用于每个数组元素，两者区别参考`$@`与`$*`的区别#TODO

${name%expr}
${name%%expr} # 删除name值中符合匹配的后缀，其必须从后缀开始就匹配
  # `%`最短匹配，`%%`最长匹配
  # 返回截取后的结果，name不变
  # 若name为`@`或`*`时操作将依次应用于每个位置参数
  # 若name为数组且下标为`[@]`或`[*]`时操作将依次应用于每个数组元素，两者区别参考`$@`与`$*`的区别#TODO
    # 例
    ( name=a1b2c3d4; echo ${name:2:3} ${name: -5:3} ${name:2:-2} ${name: -5:-1} ) # b2c 2c3 b2c3 2c3d
    ( a=a1a1b2; echo ${a#*a1} ${a##*a1} ) # a1b2 b2
    ( a=a1b2b2; echo ${a%b2*} ${a%%b2*} ) # a1b2 a1

# 扩展/参数扩展/替换
${name/expr/str} # 匹配替换，name值与expr的最长匹配将被替换为str
  # 若str为空则删除对应匹配，此时可省略/str
  # expr以`/`开头则替换所有匹配，否则只替换一次
  # 以`#`开头则必须从前缀开始就匹配，以`%`开头则必须从后缀开始就匹配
  # 若name为`@`或`*`时操作将依次应用于每个位置参数
  # 若name为数组且下标为`[@]`或`[*]`时操作将依次应用于每个数组元素#TODO

${name^expr}
${name^^expr} # 符合expr的匹配将替换为大写
  # `^`替换一次，`^^`替换多次
  # expr应确保最多只匹配一个字符，若expr省略则类似`?`匹配任意字符
  # 若name为`@`或`*`时操作将依次应用于每个位置参数
  # 若name为数组且下标为`[@]`或`[*]`时操作将依次应用于每个数组元素#TODO

${name,expr}
${name,,expr} # 符合expr的匹配将替换为小写
  # `,`替换一次，`,,`替换多次
  # expr应确保最多只匹配一个字符，若expr省略则类似`?`匹配任意字符
  # 若name为`@`或`*`时操作将依次应用于每个位置参数
  # 若name为数组且下标为`[@]`或`[*]`时操作将依次应用于每个数组元素#TODO
    # 例
    ( n=ababc; echo ${n/a*a} ${n/ab/C} ${n//ab/C} ${n/#*b/C} ${n/%b*/C} ) # bc Cabc CCc Cc aC
    ( n=aab; echo ${n^a} ${n^^a} ${n^} ${n^^} ) # Aab AAb Aab AAB
    ( n=AAB; echo ${n,A} ${n,,A} ${n,} ${n,,} ) # aAB aaB aAB aab

# 扩展/参数扩展/格式
${name@operator} # 依据operator进行对应输出
  Q # 可以直接用作输入的格式
  E # 带有转义的格式，类似`$'xxx'`#TODO
  P # 带有其他的提示字符格式 #TODO
  A # 为赋值或声明语句，执行该语句可以重建其属性和值
  a # 为参数属性标志，类似r表示只读
    # 例
    ( declare -r a='a"b&c'; echo ${a@Q} . ${a@E} . ${a@A} . ${a@a} ) # 'a"b&c' . a"b&c . declare -r a='a"b&c' . r
    ( a='second: \D{%S}'; echo ${a@Q} . ${a@P} ) # 'second: \D{%S}' . second: 29

# 扩展/命令扩展
`cmd`
$(cmd) # 在子shell环境执行cmd并替换为命令输出而后继续执行
  # 使用`$()`的形式可以直接进行嵌套，使用反撇号则需要转义
    # 例
    echo "second: $(date +%S)" # second: 58

# 扩展/算术扩展#TODO
$((expr)) # 对算术表达式求值并替换结果
$[expr] # 已废弃的旧语法，在新版本中可能被移除
  # 逻辑判断时真返回1，假返回0#TODO
    # 例
    ( a=$((6*6)); echo $a $((3*3)) ) # 36 9
    echo $(( 3 || 0 )) $(( 3 && 0 )) # 1 0

# 扩展/进程扩展
cmd1 <(cmd2) # 将cmd2视作文件读取输入
cmd1 >(cmd2) # 将cmd2视作文件写入输出
  # 命令异步运行，其输入输出以类似文件名的形式传递给命令，使用命名管道或/dev/fd/[n]的形式
    # 例
    echo <(id) # /dev/fd/63
    source <(echo 'echo $BASHPID') # 1507 因为`source`的参数要求为文件名，故而将命令输出以类似文件名的形式传递
    { echo msg; echo err >&2; } 2> >(tee o.err) # msg err 文件o.err内容err，错误重定向至文件的同时也在终端显示

# 扩展/分词
IFS # 分词主要依据其值，默认值为`$' \t\n'`，即<space><tab><newline>
  # 默认情况下开头或结尾的<space><tab><newline>序列会被忽略，其余位置的相应字符用于分词
  # 显式空参数""或''作为空字符串传递给命令
  # 扩展后是空值的无引号隐式空参数会被删除，若加上双引号则会作为空字符串传递
  # 空参数为非空扩展的一部分时会被删除，即`-d""`视为`-d`
  # 若没有发生扩展则不会进行分词

# 扩展/路径扩展
*
?
[ # 字符中含有`* ? [`且没被引号括起则会被视为匹配
  # `*`匹配任意字符串包括空字符串，`?`匹配任意一个字符，`[...]`匹配任意括号内的字符
## --- 若`extglob`启用则可以使用扩展匹配，可使用`shopt`查看或设置相应选项
?(expr) # 匹配0或1次
*(expr) # 匹配0或多次
+(expr) # 匹配1或多次
@(expr) # 匹配1次
!(expr) # 匹配除了expr的其他情况
    # 例
    shopt | grep extglob # extglob on
    echo /usr/* # /usr/bin /usr/sbin /usr/lib /usr/lib64 ...
    echo /usr/?(s)bin . /usr/*(s)bin # /usr/bin /usr/sbin . /usr/bin /usr/sbin
    echo /usr/+(s)bin . /usr/@(s)bin . /usr/!(s)bin # /usr/sbin . /usr/sbin . /usr/bin
```

```bash
# 重定向/使用重定向符号
# 默认情况下描述符`0`为标准输入，描述符`1`为标准输出，描述符`2`为标准错误
[n]<input     # 重定向输入至文件input
[n]>output    # 重定向输出至文件output
[n]>>output   # 追加模式重定向输出至文件output

&>output
>&output
>output 2>&1  # 重定向标准输出和标准错误至文件output，推荐使用`&>`语法

&>>output
>>output 2>&1 # 追加模式重定向标准输出和标准错误至文件output

# 重定向/使用`exec`#TODO
exec [n]<input   # 创建描述符n([n]为数字且最好大等于10)用于输入重定向至文件input
exec [n]>output # 创建描述符n([n]为数字且最好大等于10)用于输出重定向至文件output
## ---
exec {name}<input  # 创建描述符(`$name`为自动分配和赋值的大等于10的数字)用于输入重定向至文件input
exec {name}<&-     # 关闭对应的输入描述符
exec {name}>output # 创建描述符(`$name`为自动分配和赋值的大等于10的数字)用于输出重定向至文件output
exec {name}>&-     # 关闭对应的输出描述符
## ---
exec [n]<&[fd]  # 将输入描述符[fd]复制为描述符[n]，[n]默认为0
exec [n]<&-     # 关闭输入描述符[n]
exec [n]>&[fd]  # 将输出描述符[fd]复制为描述符[n]，[n]默认为1
exec [n]>&-     # 关闭输出描述符[n]
exec [n]<&[fd]- # 将输入描述符[fd]复制为描述符[n]之后关闭描述符[fd]，[n]默认为0
exec [n]>&[fd]- # 将输出描述符[fd]复制为描述符[n]之后关闭描述符[fd]，[n]默认为1
exec [n]<>inout # 同时用于输入输出
    # 例
    ( exec {o_fd}>/tmp/file; ls -l /dev/fd/$o_fd; echo str >&$o_fd; cat /tmp/file ) # l-wx. /dev/fd/10->/tmp/file str
    ( exec {o_fd}>/tmp/file; exec {o_fd}>&-; ls -l /dev/fd/$o_fd ) # No such file or directory
    ( exec 11</tmp/file; exec 22<&11-; ls -l /dev/fd/{11,22} ) # lr-x. "No such file or directory" /dev/fd/22->/tmp/file
```

```bash
# Here Documents
# 读取输入直到出现只有指定结束符的行(不能有空格)，其读取的所有行作为命令的标准输入
# 若eof被引号括起则内容不会执行扩展，而是原样输出
# 若eof没有被引号括起或转义，其内容string将会执行扩展，故而"` $ \"等需要转义
# 若重定向操作符是`<<-`则输入行和结束符行的前导tab将会删除，用于脚本内部的代码对齐，仅对tab有效，空格不生效
[n]<<eof
string
eof
## ---
[n]<<-eof
string
eof
  # 例
cat <<E
$(pwd) # /root
E
cat <<\E
$(pwd) # $(pwd)
E
cat <<"E"
$(pwd) # $(pwd)
E

# Here Strings
# 为Here Documents的一种变体，将内容string扩展后作为一个字符串并附加一个换行然后作为命令的标准输入
[n]<<<string
    # 例
    cat <<<~ftp # /var/ftp
    ( read a <<<'str'; echo $a ) # str
```

```bash
# 别名
# 若命令的第一个词没有括起或转义，则会检查其是否有别名，若有则替换为设置的别名值
# 替换后的别名值的第一个词会再次检查是否有别名，若该词与正在扩展的别名相同则不会再次扩展
# 若替换后的别名值最后字符是空，则会检查其后的下个词是否有别名
# bash在执行输入行的命令前至少读取一行完整输入，而别名在读取命令时展开而不是执行时，因此同一行的命令不受新别名影响，别名将在下一行生效
# 函数同上，别名在读取函数定义时扩展，而不是函数执行时，函数中定义的别名在函数执行前不可用，函数定义后新别名在函数中不生效
# 没有使用参数的机制，若需要参数则应使用函数
# 不是交互式shell不会展开别名，除非设置了`expand_aliases`
alias # 设置别名 #TODO
unalias # 取消别名
    # 例
    ( alias echo='echo 1'; eval echo ) # 1 设置别名后执行`echo`即为执行`echo 1`，单词相同故而不会递归扩展echo #TODO
    ( a(){ echo; }; alias echo='echo 1'; eval 'a; echo' ) # "" 1 后定义的别名不在函数生效
    ( alias echo='echo 1'; eval 'a(){ echo; }; a') # 1 先定义的别名生效
```

```bash
# 函数
# 函数在当前上下文执行，与执行shell脚本相比没有创建新的进程
# 函数接收的参数在执行过程中成为位置参数，`$#`会同步变化，但`$0`不变
# 局部变量仅在函数、子函数及其调用函数中可见
# shell使用动态范围控制变量可见性，因此函数看到的变量值取决于调用者对应的值
# `unset`同上，若有本地变量则删除本地变量，若没有本地变量则会向上查找引用进行删除
# `return [n]`用于函数内结束执行以及返回状态码，默认返回状态为最后一个命令的退出状态
```

```bash
# 算术表达式
# 常用格式`(( expr ))`、`$(( expr ))`、`let expr`
# 表达式计算以固定宽度的整数进行，不进行溢出检查，但除以0会报错，运算符优先级与C语言相同
# 可以将变量作为操作数，在表达式计算前执行参数扩展，可以直接使用变量而无需参数展开语法，未设置或空的变量在按名称引用时为0
# 前导`0`视为8进制数，前导`0x`或`0X`视为16进制数，可使用`[base#]n`指定进制(进制在2-64之间)，默认10进制
# 设定进制而需要字母时的顺序按小写，大写，`@`，`_`的顺序，若进制小等于36则可以互换大小写表示10-35之间的数
# 运算符按优先级计算，括号中的子表达式将优先进行计算
# 逻辑计算中0为假，非0为真
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
expr?expr:expr # 条件运算
= *= /= %= += -= <<= >>= &= ^= |= # 赋值
expr1 , expr2  # 逗号
    # 例
    ( a=1; echo $(( a++<<3 )) ) # 8
    ( a=1; echo $(( ++a<<3 )) ) # 16
    echo $(( 2#101 )) $(( 2#101 & 2#000 )) $(( 2#101 | 2#000 )) # 5 0 5
    echo $(( 0?9:3 )) # 3
    (( 0 || 3 )) && echo T; (( 0 && 3 )) || echo F # T F
    echo $(( 0 || 3 )) $(( 0 && 3 )) # 1 0
```

```bash
# 条件表达式
# 常用格式`[[ cond ]]`、`[ cond ]`、`test cond`
-a # 文件存在，为真
-b # 文件存在且为块设备，为真
-c # 文件存在且为字符设备，为真
-d # 文件存在且为目录，为真
-e # 文件存在，为真
-f # 文件存在且为普通文件，为真
-g # 文件存在且设置了sgid，为真
-h # 文件存在且为链接文件，为真
-k # 文件存在且设置了sticky，为真
-p # 文件存在且为命名管道，为真
-r # 文件存在且为可读的，为真
-s # 文件存在且尺寸大于0，为真
-t # 描述符已打开且指向终端，为真
-u # 文件存在且设置了suid，为真
-w # 文件存在且为可写的，为真
-x # 文件存在且为可执行的，为真
-G # 文件存在且所有者为egid，为真
-L # 文件存在且为链接文件，为真
-N # 文件存在且自上次读取后已被修改，为真
-O # 文件存在且所有者为euid，为真
-S # 文件存在且为套接字，为真
-ef  # 文件1和文件2引用相同设备和inode号，为真
-nt  # 文件1(修改日期)比文件2新，或文件1存在文件2不存在，为真
-ot  # 文件1(修改日期)比文件2旧，或文件2存在文件1不存在，为真
-o # shell属性被启用，为真
-v # shell变量被赋值，为真
-R # shell变量为nameref且被赋值，为真
-z # 字符串的长度为0，为真
-n  # 字符串的长度不为0，为真
str # 同上
== # 字符串1与字符串2相同(在`[[ cond ]]`中将执行模式匹配)，为真
=~ # 字符串1与字符串2匹配(字符串2被视为扩展正则表达式、仅在`[[ cond ]]`中使用)，为真#TODO
=  # 字符串1与字符串2相同，为真
!= # 字符串1与字符串2不同，为真
<  # 字典排序中字符串1在字符串2之前，为真
>  # 字典排序中字符串1在字符串2之后，为真
## --- 下述操作符左右视为整数，在`[[ cond ]]`中视为算术表达式#TODO
-eq # 参数1与参数2相等，为真
-ne # 参数1与参数2不等，为真
-lt # 参数1小于参数2，为真
-le # 参数1小等于参数2，为真
-gt # 参数1大于参数2，为真
-ge # 参数1大等于参数2，为真
```

```bash
# 命令/展开
# 分配语句(var=xxx cmd)和重定向后处理
# 扩展剩余字词，扩展后若还有字词则第一个为命令，其余为参数
# 处理重定向#TODO
# 分配语句赋值前先进行扩展
# 若没有产生命令，变量分配影响当前环境；否则变量添加至命令执行环境且不影响当前环境
# 若没有产生命令，重定向执行且不影响当前环境

# 命令/执行
# 若命令不含斜杠，查找同名函数执行，若未找到则查找内置同名函数执行
# 若都未找到，查找哈希表同名命令执行，若未找到则对`$PATH`完整查找同名命令执行
# 若都未找到，若有`command_not_found_handle`函数，则在单独环境调用且以命令及参数为该函数参数并返回对应退出状态；若无该函数则输出错误信息并返回退出状态127
# 若查找成功或命令包含斜杠，将在单独的环境执行命令，参数0设置为给定的名称，其余为函数参数
#TODO
```

---
匀速输出字符

```bash
echo "0ABC1ABC2一只敏捷的棕色的狐狸跳过一只懒惰的狗" | pv -qL 6 # 见($:pv_o)
```

---
键盘背光控制

```bash
xset led named 'Scroll Lock' # 开，见($:xset_o_led)
xset -led named 'Scroll Lock' # 关
```

---
`xset`设置用户偏好选项

```bash
# 安装
apt install x11-xserver-utils
yum install xorg-x11-server-utils
https://www.x.org/wiki

# 选项
led # (#:xset_o_led)控制键盘LED状态
    # 使用前中横线`-`关闭对应id的LED；使用关键字`named`操作对应名称的LED
    # 例
    xset led 1 # 打开led #1
    xset -led named 'Scroll Lock' # 关闭"Scroll Lock"
```

---
`rev`颠倒所输入的字符

```bash
# 安装
apt/yum install util-linux
https://www.kernel.org/pub/linux/utils/util-linux
```

---
`lolcat`彩色输出字符

```bash
# 安装
apt install lolcat
https://github.com/busyloop/lolcat
```

---
`pv`通过管道监控数据进度

```bash
# 安装
apt/yum install pv
https://www.ivarch.com/programs/pv.shtml

# (#:pv_o)选项
-q, --quiet # 不输出进度信息；在使用-L选项限制管道速率时很有用
-L, --rate-limit # 限制每秒最大的传输字节数；可使用`K, M, G, T`等后缀(*1024)
```

---
`figlet`终端输出大号文字

```bash
# 安装
apt/yum install figlet
http://www.figlet.org
```

---
`toilet`终端输出大号文字，输出带彩色

```bash
# 安装
apt install toilet
http://caca.zoy.org/wiki/toilet
```

---
`cheat`交互式备忘单

```bash
# 安装
https://github.com/cheat/cheat
```

---
`asciiview`终端查看图片

```bash
# 安装
apt install aview imagemagick
http://aa-project.sourceforge.net/aview
https://www.imagemagick.org
```

---
`cacaview`或`img2txt`终端查看图片，输出带彩色

```bash
# 安装
apt/yum install caca-utils
http://caca.zoy.org/wiki/libcaca
```

---
`sl`一辆火车通过终端

```bash
# 安装
apt/yum install sl
https://github.com/mtoyoda/sl
```

---
`cmatrix`终端字符雨

```bash
# 安装
apt install cmatrix
https://github.com/abishekvashok/cmatrix
```

---
`tr`变换、压缩、删除字符

```bash
# 安装
apt/yum install coreutils
https://www.gnu.org/software/coreutils/tr

# 选项
-c, -C, --complement # 使用指定字符集的补集
-d, --delete # 删除指定字符集
-s, --squeeze-repeats # 使用指定字符集替换每个重复序列，若指定字符集连续则压缩为一个
-t, --truncate-set1 # 先用第二个字符集的长度对第一个字符集进行截取
    # 例
    echo a1b2a3b4 | tr -s ab A # A1A2A3A4 其中a和b都替换为A
    echo a1b2a3b4 | tr -s ab AB # A1B2A3B4 其中a替换为A，b替换为B
    echo aa1aa2 | tr -s a A # A1A2 其中a替换为A，连续则压缩为一个A
    echo "a b  c   d" | tr -s ' ' ' ' # "a b c d" 将不定长度的空格压缩为1个
```
