MegaCLI的说明
===

Ubuntu12.02安装MegaCLI
---
Ubunut中有如下两种方法来安装。
1. 修改APT源
首先修改apt源，然后安装。
- 修改`/etc/apt/sources.list`，增加 `deb http://hwraid.le-vert.net/ubuntu precise main`
- 增加apt-key：`wget -O - http://hwraid.le-vert.net/debian/hwraid.le-vert.net.gpg.key | sudo apt-key add -`
- apt-get update
- apt-get install megacli megactl megaraid-status

2. 下载zip包安装
- 下载[MegaCLI](http://www.lsi.com/support/Pages/download-results.aspx?keyword=megacli)
- 解压`unzip MegaCli.zip`
- 进入相应目录后，运行`rpm2cpio MegaCli-*.rpm | cpio -idmv`，即安装完成。

查看Raid
---
命令如下：`opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -Lall -aALL`

参考[代码](http://tools.rapidsoft.de/perc/rs_check_raid_perc5i.class.php.txt)，raid对应关系如下：
```
$raid_level_mapping['Primary-0, Secondary-0, RAID Level Qualifier-0'] = '0';
$raid_level_mapping['Primary-1, Secondary-0, RAID Level Qualifier-0'] = '1';
$raid_level_mapping['Primary-5, Secondary-0, RAID Level Qualifier-3'] = '5';
$raid_level_mapping['Primary-1, Secondary-3, RAID Level Qualifier-0'] = '10';
```


