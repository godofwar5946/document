# PVE使用指南

**Proxmox Virtual Environment** is an open source server virtualization management solution based on QEMU/KVM and LXC. You can manage virtual machines, containers, highly available clusters, storage and networks with an integrated, easy-to-use web interface or via CLI.

**Proxmox Virtual Environment** 是基于 QEMU/KVM 和 LXC 的开源服务器虚拟化管理解决方案。您可以使用集成的、易于使用的 Web 界面或通过 CLI 管理虚拟机、容器、高可用性集群、存储和网络。



## 虚拟机磁盘大小控制

### 扩容磁盘(两种方式)

**通过web管理页面扩容（方式一）**

1.通过[http://{ip}:8006](http://{ip}:8006)进入web管理页面

2.选中磁盘扩容

![图片](imgs\PVE使用指南\选中硬盘.jpg)





![图片](imgs\PVE使用指南\选择扩容.jpg)



3.输入扩容大小（GiB）

![图片](imgs\PVE使用指南\扩容大小.jpg)



4.点击resize disk ，完成！



**通过命令行扩容（方式二）**

1.选择PVE宿主机，点击Shell，打开命令行

![图片](H:\java\javademos\document\imgs\PVE使用指南\选择PVE虚拟机.jpg)



2.输入 **lvs** 命令查询硬盘

```bash
root@pve:~# lvs
  LV            VG  Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data          pve twi-aotz-- <348.82g             0.53   0.50                            
  root          pve -wi-ao----   96.00g                                                    
  swap          pve -wi-ao----    8.00g                                                    
  vm-101-disk-0 pve Vwi-aotz--   31.00g data        2.80                                   
  vm-102-disk-0 pve Vwi-a-tz--    4.00m data        14.06                                  
  vm-102-disk-1 pve Vwi-a-tz--   50.00g data        0.00                                   
  vm-102-disk-2 pve Vwi-a-tz--    4.00m data        1.56                                   
  vm-103-disk-0 pve Vwi-aotz--    1.00g data        98.93  
```

3.输入 **lvdisplay** 命令查询硬盘路径

```bash
root@pve:~# lvdisplay
  --- Logical volume ---
  LV Name                data
  VG Name                pve
  LV UUID                MhiITG-ooj8-N4vR-ifjH-HkeI-02Zz-rCKweW
  LV Write Access        read/write (activated read only)
  LV Creation host, time proxmox, 2024-11-03 10:56:07 +0800
  LV Pool metadata       data_tmeta
  LV Pool data           data_tdata
  LV Status              available
  # open                 0
  LV Size                <348.82 GiB
  Allocated pool data    0.53%
  Allocated metadata     0.50%
  Current LE             89297
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:5
    
  --- Logical volume ---
  LV Path                /dev/pve/vm-102-disk-0
  LV Name                vm-102-disk-0
  VG Name                pve
  LV UUID                tazLNE-NZ8U-Rxhf-Yj7D-08Nr-OjeR-0LE2QS
  LV Write Access        read/write
  LV Creation host, time pve, 2024-11-03 12:17:22 +0800
  LV Pool name           data
  LV Status              available
  # open                 0
  LV Size                4.00 MiB
  Mapped size            14.06%
  Current LE             1
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:9
   
  --- Logical volume ---
  LV Path                /dev/pve/vm-102-disk-1
  LV Name                vm-102-disk-1
  VG Name                pve
  LV UUID                dXcssF-ka6D-XhNk-6Vom-lSKC-Nv4f-P3vUr1
  LV Write Access        read/write
  LV Creation host, time pve, 2024-11-03 12:17:22 +0800
  LV Pool name           data
  LV Status              available
  # open                 0
  LV Size                50.00 GiB
  Mapped size            0.00%
  Current LE             12800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:10
```

4.使用 **lvextend** 命令扩容硬盘

```bash
root@pve:~# lvextend -L +10G /dev/pve/vm-102-disk-1
  Size of logical volume pve/vm-102-disk-1 changed from 50.00 GiB (12800 extents) to 60.00 GiB (15360 extents).
  Logical volume pve/vm-102-disk-1 successfully resized.
```



### 缩小磁盘

**通过命令行缩小**

前三步与扩容相同，先找到要操作的硬盘路径，然后使用 **lvreduce** 命令缩小磁盘

```bash
root@pve:~# lvreduce -L -10G /dev/pve/vm-102-disk-1
  WARNING: Reducing active logical volume to 50.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
```



