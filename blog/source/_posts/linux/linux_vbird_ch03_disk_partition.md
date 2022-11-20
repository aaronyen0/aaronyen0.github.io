---
title: 鳥哥私房菜 ch3：主機規劃與磁碟分割
date: 2022-11-19 00:00:00
author: Aaron
top: false
cover: true 
toc: true
summary: Linux Disk Partition
categories: 
  - Linux Bootcamp
tags: 
  - Linux
---


## 1 各硬體裝置在Linux中的檔名

|裝置|裝置在Linux內的檔名|
|----|----|
|IDE硬碟機 | /dev/hd[a-d] |
|SCSI/SATA/USB硬碟機 | /dev/sd[a-p] |
|USB快閃碟 | /dev/sd\[a-p\](與SATA相同) | 
|軟碟機 | /dev/fd[0-1] |
|印表機 | 25針:/dev/lp[0-2], USB:/dev/usb/lp[0-15] |
|滑鼠 | USB:/dev/usb/mouse[0-15], PS2:/dev/psaux |
|當前CDROM/DVDROM | /dev/cdrom |
|當前的滑鼠 | /dev/mouse |
|磁帶機 | IDE:/dev/ht0, SCSI:/dev/st0 |


## 2 磁碟分割
### 2.1 磁碟連接的方式與裝置檔名的關係

個人電腦常見的磁碟介面有兩種， 分別是IDE與SATA介面，目前(2009)的主流已經是SATA介面了，但是老一點的主機其實大部分還是使用IDE介面。 我們稱呼可連接到IDE介面的裝置為IDE裝置，不管是磁碟還是光碟設備

再以SATA介面來說，由於SATA/USB/SCSI等磁碟介面都是使用SCSI模組來驅動的， 因此這些介面的磁碟裝置檔名都是/dev/sd\[a-p\]的格式。 但是與IDE介面不同的是，SATA/USB介面的磁碟根本就沒有一定的順序，那如何決定他的裝置檔名呢？ 這個時候就得要根據Linux核心偵測到磁碟的順序了


### 2.2 磁碟的組成複習
![hard disk](/images/linux_bootcamp/harddisk.jpg)

磁碟盤上面又可細分出磁區(Sector)與磁柱(Cylinder)兩種單位， 其中磁區每個為512 bytes。

整顆磁碟的第一個磁區特別的重要，因為他記錄了兩個重要的資訊：
- 主要開機記錄區(Master Boot Record, MBR)：可以安裝開機管理程式的地方，有446 bytes
- 分割表(partition table)：記錄整顆硬碟分割的狀態，有64 bytes


### 2.3 磁碟分割表(partition table)

![partition](/images/linux_bootcamp/partition-1.png)
在分割表所在的64 bytes容量中，總共分為四組記錄區，每組記錄區記錄了該區段的啟始與結束的磁柱號碼。

假設上面的硬碟裝置檔名為/dev/hda時，那麼這四個分割槽在Linux系統中的裝置檔名如下所示， 重點在於檔名後面會再接一個數字，這個數字與該分割槽所在的位置有關喔！
- P1:/dev/hda1
- P2:/dev/hda2
- P3:/dev/hda3
- P4:/dev/hda4


由於分割表就只有64 bytes，最多只能容納四筆分割的記錄，這四個分割的記錄被稱為主要(Primary)或延伸(Extended)分割槽。根據上面的圖示與說明，我們可以得到幾個重點資訊：
- "分割"只是針對那個64 bytes的分割表進行設定而已
- 硬碟預設的分割表僅能寫入四組分割資訊
- 這四組分割資訊我們稱為主要(Primary)或延伸(Extended)分割槽
- 分割槽的最小單位為磁柱(cylinder)
當系統要寫入磁碟時，一定會參考磁碟分割表，才能針對某個分割槽進行資料的處理

為啥要分割啊？
- 資料安全性
- 系統效能考量

延伸分割，既然第一個磁區所在的分割表只能記錄四筆資料， 那我可否利用額外的磁區來記錄更多的分割資訊？

![partition2](/images/linux_bootcamp/partition-2.png)

- 延伸分割的目的是使用額外的磁區來記錄分割資訊，延伸分割本身並不能被拿來格式化
- 上圖右下方那個區塊有繼續分割出五個分割槽， 這五個由延伸分割繼續切出來的分割槽，就被稱為邏輯分割槽(logical partition)，前面四個號碼都是保留給Primary或Extended用的，邏輯分割槽的裝置名稱號碼就由5號開始
  - P1:/dev/hda1
  - P2:/dev/hda2
  - L1:/dev/hda5
  - L2:/dev/hda6
  - L3:/dev/hda7
  - L4:/dev/hda8
  - L5:/dev/hda9


總結
- 主要分割與延伸分割最多可以有四筆(硬碟的限制)
- 延伸分割最多只能有一個(作業系統的限制)
- 邏輯分割是由延伸分割持續切割出來的分割槽；
- 能夠被格式化後，作為資料存取的分割槽為主要分割與邏輯分割。延伸分割無法格式化；
- 邏輯分割的數量依作業系統而不同，在Linux系統中，IDE硬碟最多有59個邏輯分割(5號到63號)， SATA硬碟則有11個邏輯分割(5號到15號)。


### 2.4 開機流程與主要開機記錄區(MBR)
- CMOS是記錄各項硬體參數且嵌入在主機板上面的儲存器
- BIOS則是一個寫入到主機板上的一個韌體(韌體就是寫入到硬體上的一個軟體程式)
  - 目的是在載入(load)核心檔案
- MBR：第一個可開機裝置的第一個磁區內的主要開機記錄區塊，內含開機管理程式
- 開機管理程式(boot loader)：一支可讀取核心檔案來執行的軟體
  - 提供選單：使用者可以選擇不同的開機項目，這也是多重開機的重要功能
  - 載入核心檔案：直接指向可開機的程式區段來開始作業系統
  - 轉交其他loader：將開機管理功能轉交給其他loader負責

### 2.5 Linux安裝模式下，磁碟分割的選擇(極重要)

Linux系統使用的是目錄樹架構，但是我們的檔案資料其實是放置在磁碟分割槽當中的， 問題是『如何結合目錄樹的架構與磁碟內的資料』呢？ 這個時候就牽扯到『掛載(mount)』！

- 檔案系統與目錄樹的關係(掛載)
  - 所謂的『掛載』就是利用一個目錄當成進入點，將磁碟分割槽的資料放置在該目錄下；也就是說，進入該目錄就可以讀取該分割槽

![mount](/images/linux_bootcamp/dir-3.png)


- distributions安裝時，掛載點與磁碟分割的規劃
  - 初次接觸Linux：只要分割『 / 』及『swap』即可
  - 預留一個備用的剩餘磁碟容量
  - 選擇Linux安裝程式提供的預設硬碟分割方式


### 安裝Linux前的規劃
- 選擇適當的distribution
- 主機的服務規劃與硬體的關係
  - 打造Windows與Linux共存的環境
  - NAT(達成IP分享器的功能)
    - 網路流量會比較大一點。 此時Linux主機的網路卡就需要比較好些的配備。其他的CPU、RAM、硬碟等等的影響就小很多。 
  - SAMBA(加入Windows網路上的芳鄰)
    - 分享的資料量較大，/home這個目錄可以考慮獨立出來
  - Mail(郵件伺服器)
    - CentOS一安裝完畢就提供了Sendmail及Postfix兩種mail server
    - mail server上面，重要的也是硬碟容量與網路卡速度，在此情境中，也可以將/var目錄獨立出來，並加大容量
  - Web(WWW伺服器)
    - CentOS使用的是Apache這套軟體來達成WWW網站的功能，在WWW伺服器上面，如果你還有提供資料庫系統的話， 那麼CPU的等級就不能太低，而最重要的則是RAM了！要增加WWW伺服器的效能，通常提升RAM是一個不錯的考量
  - DHCP(提供用戶端自動取得IP的功能)
  - Proxy(代理伺服器)
    - 這也是常常會安裝的一個伺服器軟體，尤其像中小學校的頻寬較不足的環境下， Proxy將可有效的解決頻寬不足的問題！但是，這個伺服器的硬體要求可以說是相對而言最高的，他不但需要較強有力的CPU來運作，對於硬碟的速度與容量以及網路卡的要求都很高
  - FTP
    - 硬碟容量與網路卡好壞相關性較高



## Linux Bootcamp: Disk Management
```shell
fdisk -l # list all disks

fdisk /dev/sdb # interactive command: to do partition(GPT, MBR) on sdb

mkfs # create file system on a partition
mkfs -t TYPE DEVICE
mkfs -t ext3 /dev/sdb2
mkfs -t ext4 /dev/sdb3
mkfs.ext4 /dev/sdb3

# mounting disk
# mount DEVICE MOUNT_POINT 
mount /dev/sdb3 /opt
mount # show current mounting(show both physical filesystem and also virtual filesystem)
df -h # disk free: report file system usage

# to make mounting permanent, add command into /etc/fstab file

# to undo mount
# umount DEVICE_OR_MOUNT_POINT
umount /dev/sdb3
umount /opt

# swap
mkswap /dev/sdb1  # prepare swap space
swapon /dev/sdb1  # enable swap partition
swapon -s  # show usage swap space

# viewing labels and UUIDs
lsblk -f

# labeling a file system
e2label /dev/sdb3 /opt
```