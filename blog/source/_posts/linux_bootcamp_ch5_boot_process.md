---
title: Linux Bootcamp Ch5 - Boot Process
date: 2022-10-30 00:00:00
author: Aaron
top: false
cover: true 
toc: true
summary: Linux Command
categories: 
  - Linux Bootcamp
tags: 
  - Linux
---

影片過於粗略，改看[鳥哥私房菜20章 開機流程](https://linux.vbird.org/linux_basic/centos5/0510osloader-centos5.php)

今天仔細讀linux boot，原文寫得很精彩，小弟買這本書好幾年了，竟然一直都沒看


# Linux開機流程分析

## 開機流程一覽
- 載入 BIOS 的硬體資訊與進行自我測試，並依據設定取得第一個可開機的裝置；
- 讀取並執行第一個開機裝置內 MBR 的 boot Loader (亦即是 grub, spfdisk 等程式)；
- 依據 boot loader 的設定載入 Kernel ，Kernel 會開始偵測硬體與載入驅動程式；
- 在硬體驅動成功後，Kernel 會主動呼叫 init 程式，而 init 會取得 run-level 資訊；
- init 執行 /etc/rc.d/rc.sysinit 檔案來準備軟體執行的作業環境 (如網路、時區等)；
- init 執行 run-level 的各個服務之啟動 (script 方式)；
- init 執行 /etc/rc.d/rc.local 檔案；
- init 執行終端機模擬程式 mingetty 來啟動 login 程式，最後就等待使用者登入啦；

## BIOS, boot loader 與 kernel 載入
### BIOS(basic input output system) 開機自我測試與 MBR
- 載入CMOS，並進一步取得主機各種硬體設定，例如
 - CPU clock rate
 - 開機搜尋順序
 - 硬體大小種類
 - 系統時間
 - 匯流排是否啟動Plug and Play (PnP, 隨插即用裝置)
 - 各設備I/O位址
 - CPU溝通的IRQ岔斷
- 開機自我測試 (power on self test, POST)
 - 硬體偵測初始化
 - 設定Pnp裝置
 - 定義開機裝置順序
 - 開機裝置資料讀取 (Master Boot Record, MBR)
   - 不同作業系統有不同檔案系統格式，需要有一個開機管理程式(Boot Loader)來處理核心檔案載入
     - 安裝在第一個磁區(sector)，也就是MBR
   - BIOS如何讀取MBR的loader呢，畢竟每個作業系統的loder都不相同?
     -  透過硬體的INT 13中斷功能，所以只要能夠偵測到磁碟就讀取

### Boot Loader 的功能
- 最主要功能認識作業系統的檔案格式並據以載入kernel到主記憶體中去執行，每個作業系統都有自己的boot loader。
  - 既然MBR只有一個那多作業系統是怎麼做到的?
- 每個作業系統預設是會安裝一套boot loader到他自己的檔案系統中
  - Linux系統安裝時，可以選擇將boot loader安裝到MBR去，也可以選擇不安裝。
  - 選擇安裝到MBR的話，那理論上MBR與boot sector都會保有一份boot loader
- boot loader功能
  - 提供選單：使用者可以選擇不同的開機項目，這也是多重開機的重要功能
  - 載入kernel檔案：直接指向可開機的程式區段來開始作業系統
  - 轉交其他 loader：將開機管理功能轉交給其他 loader 負責
    - Windows的loader預設不具有控制權轉交的功能，所以大家才強調要先裝windows再裝linux

![linux boot](/images/linux_bootcamp/loader_menu.gif)

### 載入kernel偵測硬體與 initrd 的功能
- 載入kernel到主記憶體後，透過kernel
  - 重新驅動以及偵測硬體周邊，不見得會用BIOS得到的資訊
  - kernel位置: /boot/vmlinuz

```shell
[root@www ~]# ls --format=single-column -F /boot
config-2.6.18-92.el5      <==此版本核心被編譯時選擇的功能與模組設定檔
grub/                     <==就是開機管理程式 grub 相關資料目錄
initrd-2.6.18-92.el5.img  <==虛擬檔案系統檔！
System.map-2.6.18-92.el5  <==核心功能放置到記憶體位址的對應表
vmlinuz-2.6.18-92.el5     <==就是核心檔案啦！最重要者！
```

- 為了開發便利，linux kernel module是可以動態載入的，可想成是驅動程式，會放在/lib/modules目錄下 (記得 /lib 和 / 必須放在同一個partition內)
- 因此開機過程必須要掛載根目錄，才能進一步取得kernel module
  - 例如USB, SATA, SCSI, ...
  - 假設linux安裝在SATA磁碟上
    - 透過BIOS的INT 13取得boot loader與kernel檔案開機，之後kernel接管系統偵測硬體取得額外的驅動程式
    - 問題是kernel還沒執行SATA驅動程式前，又要如何從/lib/modules取得SATA的驅動程式? (雞生蛋 蛋生雞)
      - 透過虛擬檔案系統完成
- 虛擬檔案系統 (Initial RAM Disk)
  - 通常檔名為 /boot/initrd，能被boot loader載入記憶體中模擬一個根目錄
  - 此模擬在記憶體當中的檔案系統能夠提供一支可執行的程式，透過該程式來載入開機過程中所最需要的核心模組
    - USB, RAID, LVM, SCSI 等檔案系統與磁碟介面的驅動程式啦
    - 載入完成後，最終釋放虛擬檔案系統，並掛載實際的根目錄檔案系統，幫助核心重新呼叫 /sbin/init 來開始後續的正常開機流程
- QA: 是否沒有initrd就無法順利開機？
  - 當開機時無法掛載根目錄的情況下，此時就一定需要 initrd
    - 根目錄在特殊的磁碟介面 (USB, SATA, SCSI)
    - 檔案系統較為特殊 (LVM, RAID)
  - Linux安裝在IDE介面的磁碟上，並且使用預設的ext2/ext3檔案系統，不需要 initrd也能夠進入Linux

![linux boot](/images/linux_bootcamp/osloader-flow-initramfs.jpg)


## 第一支程式 init 及設定檔 /etc/inittab 與 runlevel
- /sbin/init 最主要的功能就是準備軟體執行的環境
  - 系統的主機名稱、網路設定、語系處理、檔案系統格式及其他服務的啟動等
  - 所有動作都會透過init設定檔，/etc/inittab來規劃
  - 而inittab內還有一個很重要的設定項目，預設的runlevel(開機執行等級)

### Run level：執行等級有哪些？
規定系統使用不同的服務來啟動，讓Linux的使用環境不同。分為7個等級，分別是：
- 0 - halt (系統直接關機)
- 1 - single user mode (單人維護模式，用在系統出問題時的維護)
- 2 - Multi-user, without NFS (類似底下的 runlevel 3，但無 NFS 服務)
- 3 - Full multi-user mode (完整含有網路功能的純文字模式)
- 4 - unused (系統保留功能)
- 5 - X11 (與 runlevel 3 類似，但加載使用 X Window)
- 6 - reboot (重新開機)


### /etc/inittab 的內容與語法
- 用(:)區隔四個欄位: [設定項目]:[run level]:[init 的動作行為]:[指令項目]
  - 設定項目: 最多四個字元，代表 init 的主要工作項目，只是一個簡單的代表說明。
  - run level: 該項目在哪些 run level 底下進行的意思。如果是 35 則代表 runlevel 3 與 5 都會執行。
  - init 的動作項目: 主要可以進行的動作項目意義有
    - initdefault: 預設的 run level 設定值
    - sysinit: 系統初始化的動作項目
    - ctrlaltdel: 代表 [ctrl]+[alt]+[del] 三個按鍵是否可以重新開機的設定
    - wait: 後面欄位設定的指令項目必須要執行完畢才能繼續底下其他的動作
    - respawn: 代表後面欄位的指令可以無限制的再生 (重新啟動)。
      - 例如 tty1 的 mingetty 產生的可登入畫面， 在你登出而結束後，系統會再開一個新的可登入畫面等待下一個登入。
  - 指令項目: 亦即應該可以進行的指令，通常是一些script 

```shell
[root@www ~]# vim /etc/inittab
id:5:initdefault:                 <==預設的 runlevel 設定, 此 runlevel 為 5 

si::sysinit:/etc/rc.d/rc.sysinit  <==準備系統軟體執行的環境的腳本執行檔

# 7 個不同 run level 的，需要啟動的服務的 scripts 放置路徑：
l0:0:wait:/etc/rc.d/rc 0    <==runlevel 0 在 /etc/rc.d/rc0.d/
l1:1:wait:/etc/rc.d/rc 1    <==runlevel 1 在 /etc/rc.d/rc1.d/
l2:2:wait:/etc/rc.d/rc 2    <==runlevel 2 在 /etc/rc.d/rc2.d/
l3:3:wait:/etc/rc.d/rc 3    <==runlevel 3 在 /etc/rc.d/rc3.d/
l4:4:wait:/etc/rc.d/rc 4    <==runlevel 4 在 /etc/rc.d/rc4.d/
l5:5:wait:/etc/rc.d/rc 5    <==runlevel 5 在 /etc/rc.d/rc5.d/
l6:6:wait:/etc/rc.d/rc 6    <==runlevel 6 在 /etc/rc.d/rc6.d/

# 是否允許按下 [ctrl]+[alt]+[del] 就重新開機的設定項目：
ca::ctrlaltdel:/sbin/shutdown -t3 -r now

# 底下兩個設定則是關於不斷電系統的 (UPS)，一個是沒電力時的關機，一個是復電的處理
pf::powerfail:/sbin/shutdown -f -h +2 "Power Failure; System Shutting Down"
pr:12345:powerokwait:/sbin/shutdown -c "Power Restored; Shutdown Cancelled"

1:2345:respawn:/sbin/mingetty tty1  <==其實 tty1~tty6 是由底下這六行決定的。
2:2345:respawn:/sbin/mingetty tty2
3:2345:respawn:/sbin/mingetty tty3
4:2345:respawn:/sbin/mingetty tty4
5:2345:respawn:/sbin/mingetty tty5
6:2345:respawn:/sbin/mingetty tty6

x:5:respawn:/etc/X11/prefdm -nodaemon <==X window 則是這行決定的！
```

- 上述inittab sciprt流程
  - 先取得 runlevel 亦即預設執行等級的相關等級 (以鳥哥的測試機為例，為 5 號)；
  - 使用 /etc/rc.d/rc.sysinit 進行系統初始化
  - 由於 runlevel 是 5 ，因此只進行『l5:5:wait:/etc/rc.d/rc 5』，其他行則略過
  - 設定好 [ctrl]+[alt]+[del] 這組的組合鍵功能
  - 設定不斷電系統的 pf, pr 兩種機制；
  - 啟動 mingetty 的六個終端機 (tty1 ~ tty6)
  - 最終以 /etc/X11/perfdm -nodaemon 啟動圖形介面啦！


## init 處理系統初始化流程 (/etc/rc.d/rc.sysinit)
- /etc/inittab 裡頭有 si::sysinit:/etc/rc.d/rc.sysinit 主要是做硬體的初始化
  - 開始載入各項系統服務之前，得先做好整個系統環境，主要利用 /etc/rc.d/rc.sysinit 這個 shell script 來設定

- /etc/rc.d/rc.sysinit 主要的工作大抵有這幾項
  - 取得網路環境與主機類型：
  - 讀取網路設定檔 /etc/sysconfig/network，取得主機名稱與預設通訊閘 (gateway) 等網路環境。
  - 測試與掛載記憶體裝置 /proc 及 USB 裝置 /sys：
    - 除掛載記憶體裝置 /proc 之外，還會主動偵測系統上是否具有 usb 的裝置
    - 若有則會主動載入usb的驅動程式，並且嘗試掛載 usb 的檔案系統。
  - 決定是否啟動 SELinux ：
    - SELinux 在此時進行一些檢測， 並且檢測是否需要幫所有的檔案重新編寫標準的 SELinux 類型 (auto relabel)。
  - 啟動系統的亂數產生器
    - 亂數產生器可以幫助系統進行一些密碼加密演算的功能，在此需要啟動兩次亂數產生器。
  - 設定終端機 (console) 字形：
  - 設定顯示於開機過程中的歡迎畫面 (text banner)；
  - 設定系統時間 (clock) 與時區設定
    - 需讀入 /etc/sysconfig/clock 設定值
  - 周邊設備的偵測與 Plug and Play (PnP) 參數的測試：
    - 根據核心在開機時偵測的結果 (/proc/sys/kernel/modprobe ) 開始進行 
      - ide, scsi, 網路, 音效 等周邊設備的偵測，以及利用以載入的核心模組進行 PnP 裝置的參數測試。
  - 使用者自訂模組的載入
    - 使用者可以在 /etc/sysconfig/modules/*.modules 加入自訂的模組，則此時會被載入到系統當中
  - 載入核心的相關設定：
    - 系統會主動去讀取 /etc/sysctl.conf 這個檔案的設定值，使核心功能成為我們想要的樣子。
  - 設定主機名稱與初始化電源管理模組 (ACPI)
  - 初始化軟體磁碟陣列：主要是透過 /etc/mdadm.conf 來設定好的。
  - 初始化 LVM 的檔案系統功能
  - 以 fsck 檢驗磁碟檔案系統：會進行 filesystem check
  - 進行磁碟配額 quota 的轉換 (非必要)：
  - 重新以可讀寫模式掛載系統磁碟：
  - 啟動 quota 功能：所以我們不需要自訂 quotaon 的動作
  - 啟動系統虛擬亂數產生器 (pseudo-random)：
  - 清除開機過程當中的暫存檔案：
  - 將開機相關資訊載入 /var/log/dmesg 檔案中。

- 在 /etc/rc.d/rc.sysinit 將基本的系統設定資料都寫好了，也將系統的資料設定完整？
  - 執行 dmesg: 可知道到底開機的過程中發生了什麼
  - 很多工作的預設設定檔，都在 /etc/sysconfig/ 當中，可以去看看


## 啟動系統服務與相關啟動設定檔 (/etc/rc.d/rc N & /etc/sysconfig)

經過 /etc/rc.d/rc.sysinit 的系統模組與相關硬體資訊的初始化後，還得要啟動系統所需要的各項「服務」，依據我們在 /etc/inittab 裡面提到的 run level 設定值，就可以來決定啟動的服務項目了。

主要是透過 /etc/rc.d/rc 這個指令來處理相關任務，本章用預設的runlevel 5，/etc/rc.d/rc 5 的意義是這樣的
- 透過外部第一號參數 ($1) 來取得想要執行的腳本目錄。亦即由 /etc/rc.d/rc 5 可以取得 /etc/rc5.d/ 這個目錄來準備處理相關的腳本程式；
- 找到 /etc/rc5.d/K??* 開頭的檔案，並進行『 /etc/rc5.d/K??* stop 』的動作；
- 找到 /etc/rc5.d/S??* 開頭的檔案，並進行『 /etc/rc5.d/S??* start 』的動作；

```shell
[root@www ~]# ll /etc/rc5.d/
lrwxrwxrwx 1 root root 16 Sep  4  2008 K02dhcdbd -> ../init.d/dhcdbd
....(中間省略)....
lrwxrwxrwx 1 root root 14 Sep  4  2008 K91capi -> ../init.d/capi
lrwxrwxrwx 1 root root 23 Sep  4  2008 S00microcode_ctl -> ../init.d/microcode_ctl
lrwxrwxrwx 1 root root 22 Sep  4  2008 S02lvm2-monitor -> ../init.d/lvm2-monitor
....(中間省略)....
lrwxrwxrwx 1 root root 17 Sep  4  2008 S10network -> ../init.d/network
....(中間省略)....
lrwxrwxrwx 1 root root 11 Sep  4  2008 S99local -> ../rc.local
lrwxrwxrwx 1 root root 16 Sep  4  2008 S99smartd -> ../init.d/smartd
....(底下省略)....
```

- 檔案具有幾個特點：
  - 檔名全部以 Sxx 或 Kxx ，其中 xx 為數字，且這些數字在檔案之間是有相關性的！
    - 舉例來說，如果要啟動 WWW 服務，先得要有網路
  - 全部是連結檔，連結到 stand alone 服務啟動的目錄 /etc/init.d/ 去
    - chkconfig 就是在負責處理這個連結檔，不需要自行處理
  - 最後一個被執行的項目是S99local，亦即是：/etc/rc.d/rc.local


## 使用者自訂開機啟動程序 (/etc/rc.d/rc.local)
在完成預設 runlevel 指定的各項服務的啟動後，如果我還有其他的動作
- 舉例來說， 我還想要寄一封 mail 給某個系統管理帳號，通知系統剛剛重新開機完畢
  - 製作一個 shell script
  - 將他寫入 /etc/rc.d/rc.local，那麼該工作就會在開機的時候自動被載入喔

## 根據 /etc/inittab 之設定，載入終端機或 X-Window 介面
完成了系統所有服務啟動後，接下來 Linux 就會啟動終端機或者是 X Window 來等待使用者登入啦
- 實際參考的項目是 /etc/inittab這段

```shell
1:2345:respawn:/sbin/mingetty tty1
2:2345:respawn:/sbin/mingetty tty2
3:2345:respawn:/sbin/mingetty tty3
4:2345:respawn:/sbin/mingetty tty4
5:2345:respawn:/sbin/mingetty tty5
6:2345:respawn:/sbin/mingetty tty6
x:5:respawn:/etc/X11/prefdm -nodaemon
```

- 上述scipt意義run level 2-5 時，都會執行/sbin/mingetty (mingetty是啟動終端機指令)
  - 而且執行六個，這也是為何我們 Linux 會提供六個純文字終端機的設定所在
-  respawn的init動作項目，代表當後面的指令被終止 (terminal) 時， init 會主動的重新啟動該項目。
   -  這也是為何exit離開後，系統還是會重新顯示等待使用者輸入的畫面
- 如果run level 5，除了六個終端機之外，init 還會執行 /etc/X11/prefdm -nodaemon指令
  - 該指令主要在啟動 X Window


## 開機過程會用到的主要設定檔： /etc/modprobe.conf, /etc/sysconfig/*
/sbin/init 開機過程會用到的設定檔則大多放置在 /etc/sysconfig/ 目錄下，另外由於核心還是需要載入一些驅動程式，此時也需要系統自訂的裝置與模組對應檔 (/etc/modprobe.conf)

### /etc/modprobe.conf
- 載入使用者自訂模組，在 /etc/sysconfig/modules/ 目錄下
  - 某些條件下我們還是得對模組進行一些參數的規劃， 此時就得要使用到 /etc/modprobe.conf
    - 舉例來說，鳥哥的 CentOS 主機的 modprobe.conf 有點像這樣：
  - 這個檔案大多在指定系統內的硬體所使用的模組
    - 通常系統是可以自行產生的
    - 如果系統捉到錯誤的驅動程式，或者是想要使用更新的驅動程式來對應相關的硬體配備時， 得要自行手動的處理這個檔案
  - 詳細可看 man modprobe.conf


### /etc/sysconfig/*
開機過程中服務的相關設定檔都是記錄在 /etc/sysconfig 目錄下
- authconfig
  - 規範使用者的身份認證的機制，包括是否使用本機的 /etc/passwd, /etc/shadow 等，以及 /etc/shadow 密碼記錄使用何種加密演算法，還有是否使用外部密碼伺服器提供的帳號驗證 (NIS, LDAP) 等。 系統預設使用 MD5 加密演算法，並且不使用外部的身份驗證機制

- clock
  - 此檔案在設定 Linux 主機的時區，可以使用格林威治時間(GMT)，也可以使用台灣的本地時間 (local)。基本上，在 clock 檔案內的設定項目ZONE所參考的時區位於 /usr/share/zoneinfo 目錄下的相對路徑中。
  - 要修改時區的話，得將 /usr/share/zoneinfo/Asia/Taipei 這個檔案複製成為 /etc/localtime 才行

- i18n
  - i18n設定一些語系的使用方面，例如最麻煩的文字介面下的日期顯示問題
  - 如果你是以中文安裝的，那麼預設語系會被選擇 zh_TW.UTF8 ，所以在純文字介面之下，你的檔案日期顯示可能就會呈現亂碼，就需要更改i18n，將LC_TIME 改成en即可

- keyboard & mouse
  - keyboard 與 mouse 就是在設定鍵盤與滑鼠的形式

- network
  - network 可以設定是否要啟動網路，以及設定主機名稱還有通訊閘 (GATEWAY) 這兩個重要資訊

- network-scripts/
  - network-scripts 裡面的檔案，主要在設定網路卡


## Run level 的切換： runlevel, init

- run level 有關的啟動其實是在 /etc/rc.d/rc.sysinit 執行完畢之後。也就是說，其實run level的不同僅是 /etc/rc[0-6].d 裡面啟動的服務不同而已
- 要每次開機都執行某個預設的 run level ，則需要修改 /etc/inittab 內的設定項目， 亦即是『 id:5:initdefault: 』裡頭的數字
- 僅只是暫時變更系統的 run level 時，則使用 init [0-6] 來進行 run level 的變更，當執行 init 3 時，系統會：
  - 先比對 /etc/rc3.d/ 及 /etc/rc5.d 內的 K 與 S 開頭的檔案
  - 在新的 runlevel 亦即是 /etc/rc3.d/ 內有多的 K 開頭檔案，則予以關閉
  - 在新的 runlevel 亦即是 /etc/rc3.d/ 內有多的 S 開頭檔案，則予以啟動
- 附註
  - init 0 能夠關機
  - init 6 能夠重新開機

```shell
[root@www ~]# runlevel
N 5
# 左邊代表前一個 runlevel ，右邊代表目前的 runlevel。
# 由於之前並沒有切換過 runlevel ，因此前一個 runlevel 不存在 (N)
```

```shell
# 將目前的 runlevel 切換成為 3 (注意， tty7 的資料會消失！)
[root@www ~]# init 3
NIT: Sending processes the TERM signal
Applying Intel CPU microcode update:        [  OK  ]
Starting background readahead:              [  OK  ]
Starting irqbalance:                        [  OK  ]
Starting httpd:                             [  OK  ]
Starting anacron:                           [  OK  ]
# 這代表，新的 runlevel 亦即是 runlevel3 比前一個 runlevel 多出了上述 5 個服務

[root@www ~]# runlevel
5 3
# 前一個是 runlevel 5 ，目前的是 runlevel 3
```





## Linux系統服務控制Command(service、systemctl)

### 列出所有服務
```shell	
systemctl list-unit-files --type service -all
service --status-all
sudo systemctl | grep running #可搭配grep顯示開機啟動或是正在執行的服務
```


### 服務管理
```shell
# 啟動服務
systemctl start <service-name>
service <service-name> start

# 關閉服務
systemctl stop <service-name>
service <service-name> stop

# 重新啟動服務
systemctl restart <service-name>
service <service-name> restart

# 查詢服務狀態
systemctl status <service-name>
service <service-name> status
```

### Linux開機啟動服務
可透過systemctl控制要在開機時啟動或是不啟動服務

```shell
# 開機啟動服務
systemctl enable <service-name>

# 開機不啟動服務
systemctl disable <service-name>
```