<!--
{
    "title": "vhd安装系统",
    "create": "2019-02-17 19:13:43",
    "modify": "2019-02-17 19:13:43",
    "tag": [
        "windows",
        "vhd"
    ],
    "info": []
}
-->

## vhd安装系统

```cmd
# rufus和iso镜像制作启动盘

# 进入安装步骤，Shift + F10

# 选择磁盘
diskpart
list disk
select disk 0 # 一般为disk0 依据情况选择

# 格式化
clean
convert gpt

# 创建分区
create part efi size 100
create part primary

# 格式化
sel part 1
format quick
sel part 2
format quick

# 创建vhd磁盘
sel part 2 # 选择数据分区，依据情况选择
assign letter f # 分配标识符
create vdisk file f:\vdisk.vhd type expandable maximum 1024 # 1024MB，单位MB fixed为固定大小 expandable为依据使用增大至最大大小

# 挂载vhd磁盘
sel vdisk file f:\vdisk.vhd
attach vdisk

# 进入安装步骤，选择vdisk磁盘，格式化，安装
# 一般启动盘disk1，数据盘disk0，vdisk盘disk2，依据情况选择

# 安装完成后使用启动盘 再次进入安装步骤 Shift + F10
diskpart
sel disk 0
sel part 2
assign letter f

# 再次 Shift + F10
cd f:\
move vdisk.vhd vdisk_origin.vhd

# Alt + Tab 返回原先cmd
create vdisk file f:\vdisk.vhd parent f:\vdisk_origin.vhd # parent指定差分盘的基准

# 差分盘完成，后续修改会写入vdisk.vhd，vdisk_origin.vhd为基础，无法使用
# 还原 删除vdisk.vhd重新创建即可
del vdisk.vhd
create vdisk file f:\vdisk.vhd parent f:\vdisk_origin.vhd

# 压缩vdisk
diskpart
sel vdisk file f:\vdisk.vhd
compact vdisk
```