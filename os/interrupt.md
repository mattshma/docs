# 中断
频繁的中断会消耗很多CPU资源，多核CPU将中断分散到不同cpu核上，能获得更好的性能。

## 定义
关于中断的说明，引用[wiki](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%96%B7)上的话:

> 中断是用以提高计算机工作效率、增强计算机功能的一项重要技术。最初引入硬件中断，只是出于性能上的考量。如果计算机系统没有中断，则处理器与外部设备通信时，它必须在向该设备发出指令后进行忙等待(Busy waiting)，反复轮询该设备是否完成了动作并返回结果。这就造成了大量处理器周期被浪费。引入中断以后，当处理器发出设备请求后就可以立即返回以处理其他任务，而当设备完成动作后，发送中断信号给处理器，后者就可以再回过头获取处理结果。这样，在设备进行处理的周期内，处理器可以执行其他一些有意义的工作，而只付出一些很小的切换所引发的时间代价。后来被用于CPU外部与内部紧急事件的处理、机器故障的处理、时间控制等多个方面，并产生通过软件方式进入中断处理（软中断）的概念。

虽然中断可以提高计算机性能，但过于密集的中断请求和响应反而会影响系统性能。

每个能发出中断请求的硬件控制器都有一条名为IRQ (Interrupt ReQuest) 的输出线，所有的IRQ线都与一个名为可编程中断控制器(PIC)的硬件电路的输入引脚相连。对于多个CPU，为充分发挥SMP体系结构的并行性，Intel引入了高级可编程控制器(APIC)。仅硬件支持中断还不够，Linux内核还必须能利用这些硬件特质。Linux kenerl 2.4后的SMP IRQ Affinity支持将不同的硬件中断请求分配到特定CPU上。

## 手动分配中断到指定CPU
通过`cat /etc/interrupt`可以查看到系统上的中断是分配到哪些CPU上的。
```
# cat  /proc/interrupts
            CPU0       CPU1       CPU2       CPU3
   0:        257          0          0          0  IR-IO-APIC-edge      timer
   9:        119          0          0          0  IR-IO-APIC-fasteoi   acpi
  91:    6613278          0          0          0  IR-PCI-MSI-edge      megasas
 101:   37250625          0          0          0  IR-PCI-MSI-edge      em2-0
 102:  299684982          0          0          0  IR-PCI-MSI-edge      em2-1
 NMI:      33548       4269       3198       3328   Non-maskable interrupts
 IWI:          0          0          0          0   IRQ work interrupts
 RES:     683876     395047     340548     367534   Rescheduling interrupts
 ERR:          0
 MIS:          0

```

以上从左到右依次是IRQ号，在各CPU上已发生的中断次数，可编程中断控制器，设备名称。IRQ号决定了被CPU处理的优先级，IRQ号越小则优先级越高。

从上可以看出，CPU0处理的中断很多，部分中断在各CPU的分配不太均衡。一般而言，中断只被CPU0处理，为平均分配请求，可借助Irqbalance来自动分发中断到各CPU上，这是一种比较省心的方式。但若启动Irqbalacne后，CPU0压力仍非常大，若考虑到kernel2.4后支持的[SMP IRQ Affinity](https://github.com/torvalds/linux/blob/9256d5a308c95a50c6e85d682492ae1f86a70f9b/Documentation/IRQ-affinity.txt)特性，可以先将irqbalance（主要功能是分发中断请求到各CPU上）服务关掉，再将部分中断从CPU0分配到其他CPU上。

/proc/irq/IRQ#/smp_affinity和/proc/irq/IRQ#/smp_affinity_list指定目标cpu用于处理相应IRQ。保存在/proc/irq/IRQ#/smp_affinity的值是一个十六进制字节掩码，代表系统中所有CPU核。如CPU0，CPU1，CPU2，……分别对应1，2，4，……若指定为0f，则表明CPU0，CPU1，CPU2，CPU3处理该中断，指定为f0，则表示CPU4，CPU5，CPU6，CPU7处理该中断。/proc/irq/IRQ#/smp_affinity_list表示处理中断的相关CPU，为十进制，若相应CPU是范围值，可通过`CPUx-CPUy`来指定，如0到32号CPU都处理该中断：`echo 0-32 > smp_affinity_list`。

### 软中断
上述说的都是硬中断，对于CPU执行指令时产生的软中断，可通过`cat /proc/softirqs`查看：

```
# cat /proc/softirqs
                CPU0       CPU1
      HI:          0          0
   TIMER:   29643815    8312063
  NET_TX:      12423          0
  NET_RX: 2971796453      16580
   BLOCK:   12795017    7912339
BLOCK_IOPOLL:          0          0
 TASKLET:         19          0
   SCHED:     822379    5189920
 HRTIMER:      54211     256997
     RCU:   29334920    8460411
```

第一列为各软中断名，如NET_TX和NET_RX高说明和网卡相关。

## Reference
- [中断和 IRQ 调节](https://access.redhat.com/documentation/zh-CN/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-cpu-irq.html)
- [IRQ-affinity.txt](https://github.com/torvalds/linux/blob/9256d5a308c95a50c6e85d682492ae1f86a70f9b/Documentation/IRQ-affinity.txt)
- [proc.txt](http://www.cyberciti.biz/files/linux-kernel/Documentation/filesystems/proc.txt)
