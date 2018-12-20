<!--
{
    "title": "linux常用命令",
    "create": "2018-12-20 17:55:53",
    "modify": "2018-12-20 17:55:53",
    "tag": [
        "shell",
        "bash",
        "linux",
        "find",
        "sed",
        "awk",
        "grep",
        "tar",
        "useradd",
        "mount"
    ],
    "info": []
}
-->

## find

```bash
find <path> [-option] [-print] [-exec] {} \;
find -iname '*.md' -print;
find -iname '*.md' -exec rm -rf {} \;
find -iname '*.md' -size +1M;
find -iname '*.md' -perm 666;
find -iname '*.md' -type f;

-name -iname # 文件名 不区分大小写文件名
-type b/d/c/p/l/f # 文件类型 块设备/目录/字符设备/管道/符号链接/普通文件
-size n # 文件大小 +size -size size
-perm # 文件权限
-user -group # 文件用户/用户组
-mtime -n +n # 文件修改时间
-atime -ctime # 文件访问时间 文件创建时间
-mmin -amin -cmin # 文件修改/访问/创建 分钟
-nouser -nogroup # 无有效用户/用户组的文件
-prune # 忽略目录
-maxdepth # 查找最大深度
```

## sed

```bash
sed [option] ['commands'] file;
sed 's/old/new/g' file;
sed -n '1p;$p' file;
sed -n '1,3p' file;
sed '/pattern/,$d' file;
sed '/pattern/aappend_word' file; # sed '/pattern/a append_word' file;
sed '/pat/i insert_word' file;
sed 's/test$/&word/g' file; # 以test结尾的行，添加word 其中&为追加 不加&为替换
sed '/www/s/^/&word/g' file; # 查找www行，行首添加word
sed -e '/www/s/^/&1/' -e 's/www/w/g' file; # sed多个指令组合
sed -e '/www/s/^/&1/; s/www/w/g' file # sed多个指令组合
sed '/^$/d;G' file; # 每行追加空行
sed 'n;d' file; # 偶数行删除
sed 'n;n;d' file; # 隔2行删1行
sed '/test/{x;p;x;G} # test行前后添加空行
sed = file | sed 'N;s/\n/\t/' # 添加行号和tab制表符
sed '/2/,/6/'p file; # 打印关键词2到关键词6之间的内容

x # 指定行号
x,y # 指定范围
/pattern/ # 匹配模式
/pattern/pattern/ # 包含两个模式的行
/pattern/,x  x,/pattern/ # 从pattern的行到x行 从x行到pattern的行
x,y! # 不包括x和y的行
r w # 从另一个文件读 写到另一个文件
y # 变换字符
q # 第一个模式匹配后退出
{} # 在定位执行的命令组
p # 打印
= # 打印文件行号
a i d c s # 之后追加 之前插入 删除 新文本替换定位文本 替换模式替换相应模式
G g # 将保持缓冲区的内容追加到模式缓冲区 ...复制到模式缓冲区
n x # 读取下一行 互换模式缓冲和保持缓冲内容
```

## awk

```bash
awk 'pattern {action}' file;
df -h | awk '{print $1}';
awk -F '[ \t:;]' '{print $1}' file; # 以空格/tab/冒号/分号分隔
awk -F : '{print $1 >> "/tmp/some.log"}' file;
awk 'NR==3,NR==5 {print}' file; # 打印3/5行
awk 'length($0)>80 {print NR}' file; # 打印长度大于80的行号
awk -v STR=hello '{print STR, $NR}' file; # 引用shell变量 -v
awk -F : 'NR>=3&&NR<=10 {print $1}' file;
awk '{sum += $1}END{print sum}' file; # 计算第一列总和
awk -F : 'NR%2==0 {next} {print NR, $1}' file;
awk -F : '{printf "%-12s %-6s %-8s\n",$1,$2,$NF}' file;
awk '{print $0}' OFS='\t' file;
awk '{if(($1>$2)||($1>$3)){print $2}else{print $1}}' file;
awk 'BEGIN{count=0}{name[count]=$1;count++}END{for(i=0;i<NR;i++){print i,name[i]}}' file;
awk '{if($9~/502|404|500|503|499/)print $1,$9}' file | sort | uniq -c | sort -nr | awk '{if($1>20)print $2}'
netstat -an|awk '/tcp/ {s[$NF]++} END {for(a in s){print a,s[a]}}'
ss -an|awk '/tcp/ {s[$2]++} END {for(a in s){print a,s[a]}}'

'' # 与shell命令区分开
{} # 命令分组
pattern # 过滤器，匹配才进行action处理
FS OFS # 分隔符，默认空格 输出分隔符
NR NF # 当前行数，从1开始  当前记录字段数
$0 $1-n # 当前记录 当前第n字段

gsub(r,s) # 在$0用s替代r
index(s,t) # 返回s中t的第一个位置
length(s) # s的长度
match(s,r) # s是否匹配r
split(s,a,fs) # 在fs上将s分成序列a
substr(s,p) # 返回s从p开始的字符串

if(cond){}else{};
while{};
do{}whiile(cond);
for(init;cond;step){};
break/continue
```

## grep

```bash
grep [-option] 'keyword' file;
grep -c 'test' file;
grep -i 'teSt' file;
grep -n 'test' file;
grep -v 'test' file'
grep 'test[53]' file;
grep '^[^test]' file;
grep '[Mm]ay' file;
grep 'K...D' file;
grep 'a\{2,3\}' file;
grep -n '^$' file;
grep -vE '^$|#' file;

-a # 以文本文件方式搜索
-c # 计算符合行次数
-i # 忽略大小写
-n # 输出行号
-v # 反向选择
-h # 多文件时不显示文件名
-l # 多文件只输出包含匹配字符的文件名
-s # 不显示不存在或无匹配的错误信息
-E # 使用egrep扩展模式
```

## tar

```bash
# tar增量备份：利用/data/bak/snapshot信息进行增量备份，每次文件名不同
tar -g /data/bak/snapshot -zcvf /data/bak/data.timestr.tgz -C /path/to/bak/dir/
# 恢复，从开始一个个解压下去
tar -g /data/bak/snapshot -zxvf /data/bak/data.timestr0.tgz -C /path/to/data/dir/
tar -g /data/bak/snapshot -zxvf /data/bak/data.timestr{1..n}.tgz -C /path/to/data/dir/

tar -zxvf file.tgz
tar -zcvf file.tgz
tar -uf all.tar logo.gif
tar -tf all.tar
tar -xf all.tar
-c # 创建
-x # 解压
-t # 查看
-f # 文件名
-z # gzip压缩
-j # bzip压缩
-v # 详细信息
-u # 更新已有文件
```

## useradd

```bash
# 添加用户&密码：
useradd <user_name> -p <密文>
useradd test -p $(openssl passwd 'pwd') && usermod -U test # 设置密码并解锁用户
```

## mount

```bash
# mount使用：
mount -o remount,rw /mnt # 重挂载
mount -t ntfs-3g /dev/sdc /mnt # 挂载ntfs
mount -t iso9660 -o loop centos.iso /mnt # 挂载iso
mount -t nfs 192.168.1.1:/data /mnt # 挂载nfs

-a # 挂载/etc/fstab所有分区
-v # 显示信息
-F # 为每个挂载启动进程，提升nfs挂载速度
-f # 模拟挂载，常用于测试
```
