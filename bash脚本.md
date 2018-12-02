<!--
{
    "title": "bash脚本",
    "create": "2018-05-16 14:30:06",
    "modify": "2018-12-02 19:40:55",
    "tag": [
        "xset",
        "zram",
        "tmpfs",
        "bash"
    ],
    "info": []
}
-->

## 背光键盘灯控制

`xset`命令是设置X-Window的实用工具

```bash
#! /bin/bash
status=/tmp/keyboard_led_status
if [[ ${1} == "on" ]] || [[ ! -e ${status} ]]; then # 使用 on 或是 当前状态未开启
    xset led named 'Scroll Lock' && echo "on" > ${status}
else
    xset -led named 'Scroll Lock' && rm -f ${status}
fi
```

## 批量重命名

```bash
for file in `ls`
do
    newfile="$i.${file##*.}"
    mv $file $newfile
    ((i++))
done
# ------------------------------
basename example.tar.gz.tar.gz # => example
FILE="example.tar.gz"
echo "${FILE%%.*}" # => example
echo "${FILE%.*}" # => example.tar
echo "${FILE#*.}" # => tar.gz
echo "${FILE##*.}" # => gz
# 在bash中可以这么写
filename=$(basename "$fullfile")
extension="${filename##*.}"
filename="${filename%.*}"
```

## 开机启用zram

可选：`sudo vim /etc/rc.local`

```bash
modprobe zram && # 加载zram模块
echo $((768*1024*1024)) > /sys/block/zram0/disksize && #设置zram大小768M（10%~25%之间）
mkswap /dev/zram0 && #启用zram
swapon -p 10 /dev/zram0 && #设置优先级zram>swap
```

## 开机挂载/tmp为tmpfs

将`/tmp`挂载为tmpfs，将内存用于应用的缓存

```bash
#编辑 /etc/fstab 添加以下内容
tmpfs    /tmp    tmpfs    defaults,size=2048M    0    0
```
