> https://www.vmware.com/resources/compatibility/search.php

## NIC

### 查看NIC

```
# 列出所有NIC

esxcfg-nics -l
```

### 查看NIC设备

```
# 列出 VID、DID、SVID、SSID

vmkchdev -l | grep vmnic*
```

### 查看NIC固件和驱动版本

```
# 包括了 Driver 、Version（驱动版本）、Firmeare Version（固件版本）

esxcli network nic get -n vmnic8
```

![image-20240428163559232](C:\Users\OK馒头\AppData\Roaming\Typora\typora-user-images\image-20240428163559232.png)

## HBA

### 查看所有HBA

```
# 会列出 SATA、SAS和FC的hba

esxcfg-scsidevs -a or esxcli storage core adapter list
```

### 查看FC HBA的各种ID

```
# 从上面列出的vmhba中找到fc hba

vmkchdev -l | grep vmhba2
```

### 查看FC HBA的详细信息

```
esxcli storage san fc list
```

![image-20240428163537901](C:\Users\OK馒头\AppData\Roaming\Typora\typora-user-images\image-20240428163537901.png)

## RAID

### 查看RAID驱动

```
esxcfg-scsidevs -a
```

### 查看RAID驱动

```
esxcli storage san sas list
```

![image-20240428164224962](C:\Users\OK馒头\AppData\Roaming\Typora\typora-user-images\image-20240428164224962.png)

### 查看RAID固件

```
# 此方法可以查看所有驱动和版本

esxcli software vib list | grep lsi-mr3
```

