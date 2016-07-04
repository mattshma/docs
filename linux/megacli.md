# MegaCli

## 安装

有如下两种方法来安装。

1.Ubuntu系统通过apt-get安装
 - 修改`/etc/apt/sources.list`，增加 `deb http://hwraid.le-vert.net/ubuntu precise main`
 - 增加apt-key：`wget -O - http://hwraid.le-vert.net/debian/hwraid.le-vert.net.gpg.key | sudo apt-key add -`
 - apt-get update
 - apt-get install megacli megactl megaraid-status

2. 下载zip包安装
 - 下载[MegaCLI](http://www.lsi.com/support/Pages/download-results.aspx?keyword=megacli)
 - 解压`unzip MegaCli.zip`
 - 进入相应目录后，运行`rpm2cpio MegaCli-*.rpm | cpio -idmv`，即安装完成。

## 简介

MegaCli命令形式如下：
```
# MegaCli <-command>   [other arguments or directives]  -a<adapter identifier>
```

一般通用参数如下：

- 适配器参数-aN。N代表适配器的ID，其从0开始计数，若要指定所有的适配器，使用ALL即可。
- 物理设备参数-PhysDrv [E:S]。E表示enclosure device ID，S表示 slot number（从0开始计数）。
- 逻辑设备参数-Lx。x代表逻辑设备ID，从0开始计数，若要指定所有的逻辑设备，使用ALL。

下面简单说下适配器和设备的一些常用命令，包括ID的获取方法。

### 适配器

- 查看适配器数量：`MegaCli -adpCount`
- 查看适配器ID：`MegaCli -AdpAllInfo -aAll | grep ^Adapter`
- 查看适配器时间：`MegaCli -AdpGetTime –aALL`

### 物理设备

- 查看所有物理设备信息

命令为：`MegaCli -PDList -aAll`。可以看到`Enclosure Device ID`和`Slot Number`值。

### 逻辑设备

- 查看所有逻辑设备

命令为：
```
# MegaCli -LDInfo -Lall -aALL
Adapter 0 -- Virtual Drive Information:
Virtual Drive: 0 (Target Id: 0)
Name                :
RAID Level          : Primary-1, Secondary-0, RAID Level Qualifier-0
Size                : 278.875 GB
State               : Optimal
Strip Size          : 128 KB
Number Of Drives    : 2
Span Depth          : 1
Default Cache Policy: WriteBack, ReadAhead, Direct, No Write Cache if Bad BBU
Current Cache Policy: WriteBack, ReadAhead, Direct, No Write Cache if Bad BBU
Access Policy       : Read/Write
Disk Cache Policy   : Enabled
Encryption Type     : None


Virtual Drive: 1 (Target Id: 1)
Name                :
RAID Level          : Primary-5, Secondary-0, RAID Level Qualifier-3
Size                : 40.019 TB
State               : Optimal
Strip Size          : 128 KB
Number Of Drives    : 12
Span Depth          : 1
Default Cache Policy: WriteBack, ReadAhead, Direct, No Write Cache if Bad BBU
Current Cache Policy: WriteBack, ReadAhead, Direct, No Write Cache if Bad BBU
Access Policy       : Read/Write
Disk Cache Policy   : Enabled
Encryption Type     : None  
```

Virtual Drive后的数字为逻辑设备参数。该命令也可查看系统所做的Raid，参考[代码](http://tools.rapidsoft.de/perc/rs_check_raid_perc5i.class.php.txt)，raid对应关系如下：
```
$raid_level_mapping['Primary-0, Secondary-0, RAID Level Qualifier-0'] = '0';
$raid_level_mapping['Primary-1, Secondary-0, RAID Level Qualifier-0'] = '1';
$raid_level_mapping['Primary-5, Secondary-0, RAID Level Qualifier-3'] = '5';
$raid_level_mapping['Primary-1, Secondary-3, RAID Level Qualifier-0'] = '10';
```

- 查看逻辑盘详细信息

命令为：`MegaCli -LdPdInfo -aALL`。

### 电池

命令为：`MegaCli -AdpBbuCmd -aALL`。


## Raid

- 新建Raid：`MegaCli -CfgLdAdd -r(0|1|5) [E:S, E:S, ...] -aN`。如创建RAID0:`MegaCli -CfgLdAdd -r0 [10:0] -a0`和创建Raid10:`MegaCli -CfgSpanAdd -r10 -Array0[E:S,E:S] -Array1[E:S,E:S] -aN`。

- 删除Raid：`MegaCli -CfgLdDel -Lx -aN`。

## 实战

### 找到更换的盘
系统换过磁盘后，不知道换的是哪块盘，通过`MegaCli -PDList -aALL`可以看到有块盘的状态为foreign:
```
# MegaCli -PDList -aALL
Enclosure Device ID: 22
Slot Number: 5
...
Foreign State: Foreign
...

```

则该盘为刚更换的盘，接着清除foreign状态：`MegaCli -cfgforeign -clear -a0`。然后做Raid即可：`MegaCli -CfgLdAdd -r0 [22:5] -a0`。若不清foreign就直接做raid，会报如下错误：

```
The specified physical disk does not have the appropriate attributes to complete
the requested command.

Exit Code: 0x26
```

### 换磁盘
更换磁盘后，megacli查看状态如下：
```
# MegaCli -PDlist -a0 |grep "Firmware state"
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Unconfigured(good), Spun down
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
Firmware state: Online, Spun Up
```

对于需要备份的raid级别，通过`# MegaCli -PDHSP -Set -PhysDrv [22:6] -a0`将该盘设置为Hotspare状态。若希望该盘直接使用，通过`MegaCli -CfgLdAdd -r0 [22:6] -a0`可将其调整为`Firmware state: Online, Spun Up`。
