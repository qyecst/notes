<!--
{
    "title": "shell相关",
    "create": "2018-12-20 17:55:53",
    "modify": "2018-12-20 17:55:53",
    "tag": [
        "shell",
        "bash"
    ],
    "info": []
}
-->

## shell语法

```bash
# 变量格式
var_name = var_value # 区分大小写，变量名不以数字，特殊字符开头；值可以是数字、字符串、文件位置、命令、命令结果、 .etc

# 变量分 自定义变量、环境变量、位置变量、预定义变量
echo $var_name # 变量输出
export var_name # 转为环境变量

# 环境变量
echo $PWD $HOME $HOSTNAME $BASH_VERSION $BASH_VERSINFO $GROUPS $SHELL $UID # .etc
env # 查看环境变量

# 预定义变量
$# 传递到脚本的参数个数 $* 以一个字符串显示所有参数 $$ 脚本当前进程id号 $! 后台运行的最后一个进程id $@ 返回所有参数，以""包裹每个参数
$- 显示shell使用的选项，与set命令相同 $? 最后命令的退出状态，0表示无错误

# varname=$(expr $1 + $2) 整数运算，注意运算符左右空格
expr $1 + $2 ; expr $1 - $2 ; expr $1 \* $2 ; expr $1 / $2 ; expr $1 % $2

varname="some value .etc" # ""保留值中的空格或特殊字符，''将值内的全部作为普通字符，``反撇号，同$()，建议$()，先执行命令，再赋值
read varname # 读取输入 read -p '提示信息' varname

# 条件测试 test [] [[]]
test -r file
[ -e file ]
[[ 3 -eq 3 && 4 -eq 4 ]]
# 其内变量、字符串使用双引号包围,能做字符串比较的时候，不要用数值比较 [ "$name" = "str string" ]
# 将对数值的等值比较改为字符串的比较，避免变量为空时报错 [ "$a" = "7" ]
# 变量可能为空的时候，在变量的基础上加上其他辅助字符串 [ "a$a" = "a7" ]#判断a是否为7 [ "a$a" = "a" ]#判断a是否为空
# 在[]和[[]]中每个地方都有空格，强制要求格式

# if语句格式
if [ ! -d /not/exists/dir ]; then
    <do something>
fi
---
if [ "$UNAME" == "Linux" ]; then
    <do something>
else # if [ "$UNAME" == "Darwin" ]; then
    <do something>
    if [ ! -x ${STEAMEXE} ]; then
        <do something>
    fi
    <do something>
fi

# while语句格式，可用break continue
while [ $count -le 6 ]; do
    <do something>
done

# for语句格式，可用break continue
for ((i=1;i<=10;i++)); do
    <do something>
done
---
for i in $(seq 1 10); do
    <do something>
done
---
for i in {1..10}; do
    <do something>
done
---
for i in `ls`; do
    <do something>
done
---
for i in $*; do
    <do something>
done
---
for i in f1 f2 f3; do
    <do something>
done
---
list="rootfs usr data data2"
for i in $list; do
    <do something>
done
---
for file in /proc/*; do
    <do something>
done
---
for file in $(ls *.sh); do
    <do something>
done

# case语句格式
case varname in # 起始语句，case <xxx> in
    pattern1) # 匹配模式，以右括号结束
    <do something>
    ;; # 双分号，命令序列结束标记，在bash中不加`;;`报错
    pattern2|pattern3) # |表示'或'，[0-9]表示0~9范围
    <do something>
    ;;
    *) # 默认，前面均未匹配上则执行此后命令序列
    <do something>
    ;;
esac # 结束语句

# 输出提示信息
cat <<EOF
----------------------------
请选择：
    1.显示系统信息
    2.显示磁盘使用情况
    3.显示用户空间使用情况
    0.退出
----------------------------
EOF

# 日期
date "+%Y-%m-%d %H:%M:%S" # 2018-06-24 23:48:23

# 退出状态，退出shell，并返回给定值。在shell脚本中可以终止当前脚本执行。执行exit可使shell以指定的状态值退出
exit 0|1|2 .etc
/**
0 正常退出
1 通常的位置错误
2 用法不当
126 命令无法执行
127 没有找到命令
128 无效退出参数
130 使用Ctrl + C终止的命令
*/

# shift 命令，脚本参数左移
sh a.sh 1 2 3 4 5
shift 3
sh a.sh (1 2 3)丢弃 4($1) 5($2)
shift (默认 shift 1)

# 函数 function
function func_name(){
    <do something>
    [return]
}
--- # function 关键字可省略，exit退出脚本，break中断函数执行
func_name(){
    <do something>
    [return]
}
---
# 调用函数
func_name # 不用 func_name()

# declare命令，定义变量
declare -f # 显示定义函数清单
declare -F # 显示定义函数名
unset -f # 从shell中删除函数
export -f # 将函数输出给shell
export -p # 列出当前的环境变量

# select选择
select i in (expression)
do
    ...etc
done
---
select i in ubuntu centos debian # 输入1 2 3
do
    case $i in
        ubuntu)
            echo your choose $i
            ;;
        centos)
            echo your choose $i
            ;;
        debian)
            echo your choose $i
            ;;
        *)
            break
    esac
done

# 数组
arr=(
    test1
    test2
)
arr=(test1 test2 test3)
${arr[0]} # 第一个数组变量
${arr[1]} # 第二个数组变量
${arr[@]} # 数组所有参数
${#arr[@]} # 数组参数长度
${#arr[0]} # 数组第一个变量长度
${arr[@]:0} # 数组所有值
${arr[@]:1} # 数组第二个开始的所有值
${arr[@]:0:2} # 数组第一个到第二个值 闭开区间
arr=([0]=te1 [1]=te2 [2]=te3) # 赋值
${arr[@]/test/some} # 将test替换为some
unset arr[0] # 删除第一个元素
unset arr # 删除数组

# 正则表达式
\ # 转义
. # 单个字符（除NUL）
* # 任意0~多个其前面的字符 .* 表匹配任意（多个）字符
^ # 以开头
$ # 以结尾
[] # 括号内任一字符 - 表范围 ^ 在括号头部表示不匹配括号内字符 e.g. [a-g] 匹配a~g [^a-g] 不匹配a~g [ag] 匹配a、g
? # 匹配前0-1次
{n,m} # 匹配前面n~m次，注意符号转义
() # 保留空间，最多将9个独立子模式存储，\<num>使用，(ab).*\1 等价 ab.*ab，注意符号转义
+ # 匹配前1-多次
| # 匹配 | 前或后的表达式

# cut文本处理
cut -b 字节分割 -c 字符分割 -d 自定分隔符 -f 字段分割，常与-d一起使用 -n 取消分割，仅和-b一起使用

# printf 输出
printf 'format_string' output_value
%ns # 输出字符串，n为输出位数，省略则输出全部
%ni # 输出整数
%m.nf # 输出浮点数，m n为数字，m表示输出总位数含小数点，n为小数位数 %0m.nf 表示空白填充0
\a # 警告声
\b # 退格 Backspace
\f # 清屏
\n # 换行
\r # 回车
\t # 水平tab
\v # 垂直tab

# sed文本处理
sed -n 只显示符合的行 -i 直接修改源文件 -e 多操作同时进行
sed [options] Address Command 修饰符 file1 file2
# Address:
linenumber # 特定行，$ 最后一行
start,end # 起始，结束行
/^root/ # 正则，以root开头的行
mode1,mode2 # 第一次mode1匹配的行开始，到第一次mode2匹配的行结束
start,+n # 从开始行往后n行
# Command:
d # 删除行
p # 输出行
a # 行后追加内容
i # 行前追加内容
r file # 将文件内容添加到符合的行
w file # 将文件内容另存为符合的行
s # 查找替换
# 修饰符：
g # 所有
i # 忽略大小写

# awk文本处理
awk [options] 'command' file1, file2, ...etc
```
