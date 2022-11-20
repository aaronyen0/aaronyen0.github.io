---
title: 鳥哥私房菜 ch6：檔案權限與目錄配置
date: 2022-11-20 00:00:00
author: Aaron
top: false
cover: true
toc: true
summary: Linux file permission
categories: 
  - Linux Bootcamp
tags:
  - Linux
---

## 1. 使用者與群組

- 使用者種類
  - owner, group, other
  - root

- 設定檔
  - 系統上的帳號與一般身份使用者，還有root相關資訊，記錄在: /etc/passwd
  - 個人的密碼記錄在: /etc/shadow
  - 群組名稱都紀錄在: /etc/group

## 2. Linux檔案權限概念

Linux檔案權限的重要性
- 系統保護的功能
- 團隊開發軟體或資料共用的功能
- 造成危害


```shell
[root@www ~]# ls -al
total 156
drwxr-x---   4    root   root     4096   Sep  8 14:06 .
drwxr-xr-x  23    root   root     4096   Sep  8 14:21 ..
-rw-------   1    root   root     1474   Sep  4 18:27 anaconda-ks.cfg
-rw-------   1    root   root      199   Sep  8 17:14 .bash_history
-rw-r--r--   1    root   root       24   Jan  6  2007 .bash_logout
-rw-r--r--   1    root   root      191   Jan  6  2007 .bash_profile
-rw-r--r--   1    root   root      176   Jan  6  2007 .bashrc
-rw-r--r--   1    root   root      100   Jan  6  2007 .cshrc
drwx------   3    root   root     4096   Sep  5 10:37 .gconf      <=範例說明處
drwx------   2    root   root     4096   Sep  5 14:09 .gconfd
-rw-r--r--   1    root   root    42304   Sep  4 18:26 install.log <=範例說明處
-rw-r--r--   1    root   root     5661   Sep  4 18:25 install.log.syslog
[    1   ][  2 ][   3  ][  4 ][    5   ][     6      ][       7             ]
[  權限  ][連結][擁有者 ][群組 ][檔案容量][  修改日期   ][      檔名            ]
```

1. 第一欄permission
```shell
[-][rwx][r-x][r--]
 1  234  567  890
```
   - 1：代表這個檔名為目錄或檔案，本例中為檔案(-)；
     - [ d ]則是目錄，例如上表檔名為『.gconf』的那一行；
     - [ - ]則是檔案，例如上表檔名為『install.log』那一行；
     - [ l ]則表示為連結檔(link file)；
     - [ b ]則表示為裝置檔裡面的可供儲存的周邊設備(可隨機存取裝置)；
     - [ c ]則表示為裝置檔裡面的序列埠設備，例如鍵盤、滑鼠(一次性讀取裝置)。
   - 234：擁有者的權限，本例中為可讀、可寫、可執行(rwx)；
   - 567：同群組使用者權限，本例中為可讀可執行(rx)；
   - 890：其他使用者權限，本例中為可讀(r)

2. 第二欄表示有多少檔名連結到此節點(i-node)：
每個檔案都會將他的權限與屬性記錄到檔案系統的i-node中，不過目錄樹卻是使用檔名來記錄， 因此每個檔名就會連結到一個i-node！這個屬性記錄有多少不同的檔名連結到相同的一個i-node號碼

3. 第三欄表示這個檔案(或目錄)的"擁有者帳號"
4. 第四欄表示這個檔案的所屬群組
5. 第五欄為這個檔案的容量大小，預設單位為bytes；
6. 第六欄為這個檔案的建檔日期或者是最近的修改日期：
   - 內容分別為日期(月/日)及時間。如果被修改的時間距離現在太久了，那麼時間僅顯示年份而已。
   - 如果想要顯示完整的時間格式，可以利用ls的選項，亦即：`ls -l --full-time`
   - 如果以中文安裝Linux系統，那麼日期欄位將會以中文來顯示。但中文並沒有辦法在純文字的終端機模式正確的顯示，此欄會變成亂碼。可以使用`LANG=en_US`修改語系
   - 讓系統預設的語系變成英文的話，那麼你可以修改系統設定檔`/etc/sysconfig/i18n`

7. 第七欄為這個檔案的檔名


### 2.1 Linux檔案屬性

各種身份的權限之修改的指令，如下
#### chgrp：改變檔案所屬群組
- change group
- 群組名稱必須要在/etc/group檔案內存在

```shell
[root@www ~]# chgrp [-R] 群組名 檔案或目錄, ...
參數 -R: 進行遞迴(recursive)持續變更，用在變更某一目錄內所有的檔案之情況。

假設已知在/etc/group裡面已經存在一個名為users的群組
[root@www ~]# chgrp users install.log
[root@www ~]# ls -l
-rw-r--r--  1 root users 68495 Jun 25 08:53 install.log

[root@www ~]# chgrp testing install.log
chgrp: invalid group name `testing' <== 發生錯誤訊息囉～找不到這個群組名～
```


#### chown：改變檔案擁有者
- change owner
- 使用者名稱必須要在/etc/passwd存在
- 也可以同時修改帳號及群組

```shell
[root@www ~]# chown [-R] 帳號名稱 檔案或目錄
[root@www ~]# chown [-R] 帳號名稱:群組名稱 檔案或目錄
# 參數 -R: 進行遞迴(recursive)的持續變更，亦即連同次目錄下的所有檔案都變更

# 範例：將install.log的擁有者改為bin這個帳號：
[root@www ~]# chown bin install.log

# 範例：將install.log的擁有者與群組改回為root：
[root@www ~]# chown root:root install.log
```



#### chmod：改變檔案的權限, SUID, SGID, SBIT等等的特性
- 權限的設定方法有兩種，可以使用數字或者是符號來進行權限的變更
- 數字類型改變檔案權限
  - Linux檔案的基本權限就有九個，分別是owner/group/others三種身份各有自己的read/write/execute權限`-rwxrwxrwx`
    - r:4, w:2, x:1
  - 每種身份(owner/group/others)各自的三個權限(r/w/x)分數是需要累加的，例如當權限為： `[-rwxrwx---]` 分數是
    - owner = rwx = 4+2+1 = 7
    - group = rwx = 4+2+1 = 7
    - others= --- = 0+0+0 = 0

  ```shell
  [root@www ~]# chmod [-R] xyz 檔案或目錄
  # 選項與參數：
  # xyz : 就是剛剛提到的數字類型的權限屬性，為 rwx 屬性數值的相加。
  # -R : 進行遞迴(recursive)的持續變更，亦即連同次目錄下的所有檔案都會變更

  # 如果要將.bashrc這個檔案所有的權限都設定啟用，那麼就下達：
  [root@www ~]# chmod 777 .bashrc

  # 設定 `-rwxr-xr--`
  [root@www ~]# chmod 754 filename

  # 將該檔案變成可執行檔，並且不要讓其他人修改 `-rwxr-xr-x`
  [root@www ~]# chmod 755 filename

  # 不希望被其他人看到 `-rwxr-----`
  [root@www ~]# chmod 740 filename
  ```

  
- 符號類型改變檔案權限
  - (1)user (2)group (3)others可以藉由u, g, o來代表三種身份的權限
  - a 則代表 all 亦即全部的身份，讀寫的權限就可以寫成r, w, x
  - 使用`+-`的主要用途在，沒被指定到的項目，該權限「不會被變動」

  ![chmod](/images/linux_bootcamp/chmod.png)

  ```shell
  # 一個檔案的權限成為`-rwxr-xr-x`
  # 注意！u=rwx,go=rx 是連在一起的，中間並沒有任何空白字元！
  [root@www ~]# chmod  u=rwx,go=rx  .bashrc

  # 只想要增加.bashrc這個檔案的每個人均可寫入的權限
  [root@www ~]# chmod  a+w  .bashrc

  # 將執行權限去掉而不更動其他已存在的權限
  [root@www ~]# chmod  a-x  .bashrc
  ```

### 2.3 目錄與檔案之權限意義
- 權限對檔案的重要性
  - r (read)：可讀取此一檔案的實際內容，如讀取文字檔的文字內容等；
  - w (write)：可以編輯、新增或者是修改該檔案的內容(但不含刪除該檔案)；
  - x (eXecute)：該檔案具有可以被系統執行的權限。

- 權限對目錄的重要性
  - r (read contents in directory)：表示具有讀取目錄結構清單的權限，可以利用 ls 這個指令將該目錄的內容列表顯示出來
  - w (modify contents of directory)：異動該目錄結構清單的權限，也就是底下這些權限
    - 建立新的檔案與目錄
    - 刪除已經存在的檔案與目錄(不論該檔案的權限為何！)
    - 將已存在的檔案或目錄進行更名
    - 搬移該目錄內的檔案、目錄位置

- x (access directory)：
  - 目錄不能拿來執，目錄的x代表的是使用者能否進入該目錄成為工作目錄(work directory)的用途！例如`cd`(change directory)進入該目錄
  - 如果你在某目錄下不具有x的權限， 那麼你就無法切換到該目錄下，也就無法執行該目錄下的任何指令，即使你具有該目錄的r的權限
  - 要開放目錄給任何人瀏覽時，應該至少也要給予r及x的權限，但w權限不可隨便給


### 2.4 Linux檔案種類與副檔名

#### 檔案類型
使用`ls -l`觀察到第一欄那十個字元中，第一個字元為檔案的類型。除了常見的一般檔案(-)與目錄檔案(d)之外
- 正規檔案(regular file )：第一個字元為 `[ - ]`，又大略可以分為：
  - 純文字檔(ASCII)：可以下達`cat ~/filename`就可以看到該檔案的內容
  - 二進位檔(binary)：系統其實僅認識且可以執行二進位檔案(binary file)，例如剛剛的cat就是一個binary file
  - 資料格式檔(data)：有些程式在運作的過程當中會讀取某些特定格式的檔案，那些特定格式的檔案可以被稱為資料檔 (data file)
- 目錄(directory)：第一個屬性為 `[ d ]`
- 連結檔(link)：類似Windows系統的捷徑，第一個屬性為 `[ l ]`(英文L的小寫)
- 設備與裝置檔(device)：系統周邊及儲存等相關的一些檔案，通常集中在/dev這個目錄之下，常見又分為兩種：
  - 區塊(block)設備檔 `[b]`：就是一些儲存資料，提供系統隨機存取的周邊設備，舉例來說，硬碟與軟碟等，你可以自行查一下/dev/sda看看
  - 字元(character)設備檔 `[c]`：亦即是一些序列埠的周邊設備，例如鍵盤、滑鼠等，特色就是『一次性讀取』的，不能夠截斷輸出。舉例來說，不能讓滑鼠『跳到』另一個畫面，而是『滑動』到另一個地方
- 資料接口檔(sockets)：第一個屬性為 `[ s ]`，這種類型的檔案通常被用在網路上的資料承接了。我們可以一個程式來監聽用戶端的要求，用戶端就可以透過這個socket來進行資料的溝通，最常在/var/run這個目錄中看到這種檔案類型
- 資料輸送檔(FIFO, pipe)：第一個屬性為 `[ p ]`，FIFO是一種特殊的檔案類型，主要在解決多個程序同時存取一個檔案所造成的錯誤問題

#### 副檔名
基本上，Linux的檔案是沒有所謂的副檔名的，不過我們通常我們還是會以適當的副檔名來表示該檔案是什麼種類。底下有數種常用的副檔名：

- `*.sh`：腳本或批次檔 (scripts)，因為批次檔為使用shell寫成的，所以副檔名就編成 .sh 囉；
- `*Z, *.tar, *.tar.gz, *.zip, *.tgz`：經過打包的壓縮檔。這是因為壓縮軟體分別為 gunzip, tar 等等的，由於不同的壓縮軟體，而取其相關的副檔名
- `*.html, *.php`：網頁相關檔案，分別代表 HTML 語法與 PHP 語法的網頁檔案
  - `*.html` 的檔案可使用網頁瀏覽器來直接開啟
  - `*.php` 的檔案，則可以透過 client 端的瀏覽器來 server 端瀏覽，以得到運算後的網頁結果

#### 檔案名稱限制
- 在Linux底下，使用預設的Ext2/Ext3檔案系統時，針對檔案的檔名長度限制為：
  - 單一檔案或目錄的最大容許檔名為 255 個字元
  - 包含完整路徑名稱及目錄 (/) 之完整檔名為 4096 個字元

## 3. Linux目錄配置
- Filesystem Hierarchy Standard ([FHS](http://www.pathname.com/fhs))

### 3.1 Linux目錄配置的依據--FHS

FHS依據檔案系統使用的頻繁與否與是否允許使用者隨意更動，將目錄定義成為四種交互作用的形態

|        | 可分享的(shareable) | 不可分享的(unshareable) |
|--------|--------|--------|
| 不變的(static) | /usr (軟體放置處), /opt (第三方協力軟體) | /etc (設定檔), /boot (開機與核心檔) |
| 可變動的(variable) | /var/mail (使用者郵件信箱), /var/spool/news (新聞群組) | /var/run (程序相關), /var/lock (程序相關) |

- 可分享的：可以分享給其他系統掛載使用的目錄，所以包括執行檔與使用者的郵件等資料，是能夠分享給網路上其他主機掛載用的目錄
- 不可分享的：自己機器上面運作的裝置檔案或者是與程序有關的socket檔案等，僅與自身機器有關，所以不適合分享給其他主機
- 不變的：有些資料是不會經常變動的，跟隨著distribution而不變動。例如函式庫、文件說明檔、系統管理員所管理的主機服務設定檔等
- 可變動的：經常改變的資料，例如登錄檔、一般用戶可自行收受的新聞群組等

FHS針對目錄樹架構僅定義出三層目錄底下應該放置什麼資料而已，分別是底下這三個目錄的定義：
- `/` (root, 根目錄)：與開機系統有關；
- `/usr` (unix software resource)：與軟體安裝/執行有關；
- `/var` (variable)：與系統運作過程有關。


#### 根目錄 (/) 的意義與內容：
根目錄是整個系統最重要的一個目錄，不但所有的目錄都是由根目錄衍生出來的，同時也與開機/還原/系統修復等動作有關。由於系統開機時需要特定的開機軟體、核心檔案、開機所需程式、函式庫等等檔案資料，若系統出現錯誤時，根目錄也必須要包含有能夠修復檔案系統的程式才行。因為根目錄是這麼的重要，所以在FHS的要求方面，他希望根目錄不要放在非常大的分割槽內，因為越大的分割槽會放入越多的資料，如此一來根目錄所在分割槽就可能會有較多發生錯誤的機會。

因此FHS標準建議：根目錄(/)所在分割槽應該越小越好， 且應用程式所安裝的軟體最好不要與根目錄放在同一個分割槽內，保持根目錄越小越好。 如此不但效能較佳，根目錄所在的檔案系統也較不容易發生問題

- `/bin`: 系統有很多放置執行檔的目錄，但/bin比較特殊。因為/bin放置的是在單人維護模式下還能夠被操作的指令
  - /bin底下的指令可以被root與一般帳號所使用
  - 主要有: cat, chmod, chown, date, mv, mkdir, cp, bash等等常用的指令。
- `/boot`: 主要在放置開機會使用到的檔案，包括Linux核心檔案以及開機選單與開機所需設定檔等等
  - Linux kernel常用的檔名為：vmlinuz，如果使用的是grub這個開機管理程式，則還會存在/boot/grub/這個目錄
- `/dev`: 任何裝置與周邊設備都是以檔案的型態存在於這個目錄當中的。 你只要透過存取這個目錄底下的某個檔案，就等於存取某個裝置
  - 比要重要的檔案有: /dev/null, /dev/zero, /dev/tty, /dev/lp*, /dev/hd*, /dev/sd*等等
- `/etc` (and so on): 系統主要的設定檔幾乎都放置在這個目錄內，例如人員的帳號密碼檔、各種服務的啟始檔等
  - 一般來說，這個目錄下的各檔案屬性是可以讓一般使用者查閱的，但只有root有權力修改
  - FHS建議不要放置可執行檔(binary)在這個目錄中
  - 比較重要的檔案有: /etc/inittab, /etc/init.d/, /etc/modprobe.conf, /etc/X11/, /etc/fstab, /etc/sysconfig/ 等。另外，其下重要的目錄有：
    - `/etc/init.d/`：所有服務的預設啟動 script 都是放在這裡的，例如要啟動或者關閉iptables: `/etc/init.d/iptables start`, `/etc/init.d/iptables stop`
    - `/etc/xinetd.d/`：這就是所謂的super daemon管理的各項服務的設定檔目錄。
    - `/etc/X11/`：與 X Window 有關的各種設定檔都在這裡，尤其是 xorg.conf 這個 X Server 的設定檔。
- `/home`: 這是系統預設的使用者家目錄(home directory)。在你新增一個一般使用者帳號時，預設的使用者家目錄都會規範到這裡，另外家目錄有兩種代號：
  - `~`：代表目前這個使用者的家目錄，而
  - `~dmtsai`：代表 dmtsai 的家目錄
- `/lib`: 系統的函式庫非常的多，而/lib放置的則是在"開機時會用到的函式庫"，以及在/bin或/sbin底下的指令會呼叫的函式庫而已
  - 尤其重要的是/lib/modules/這個目錄，因為該目錄會放置核心相關的模組(驅動程式)
- `/media`: media是媒體的英文，/media底下放置的就是可移除的裝置！包括軟碟、光碟、DVD等等裝置都暫時掛載於此
  - 常見的檔名有：/media/floppy, /media/cdrom等等。
- `/mnt`: 如果妳想要暫時掛載某些額外的裝置，一般建議妳可以放置到這個目錄中
- `/opt`: 這個是給第三方協力軟體放置的目錄。
  - 什麼是第三方協力軟體啊？舉例來說，KDE這個桌面管理系統是一個獨立的計畫，不過他可以安裝到Linux系統中，因此KDE的軟體就建議放置到此目錄下了。
  - 另外，如果自行安裝額外的軟體(非原本的distribution提供的)，那麼也能夠將軟體安裝到這裡來。不過以前的Linux系統中還是習慣放置在/usr/local目錄下
- `/root`: 系統管理員(root)的家目錄。因為如果進入單人維護模式而僅掛載根目錄時，該目錄就能夠擁有root的家目錄，所以我們會希望root的家目錄與根目錄放置在同一個分割槽中。
- `/sbin`: Linux有非常多指令是用來設定系統環境的，這些指令只有root才能夠利用來『設定』系統，其他使用者最多只能用來『查詢』而已。放在/sbin底下的為開機過程中所需要的，裡面包括了開機、修復、還原系統所需要的指令
  - 某些伺服器軟體程式，一般則放置到/usr/sbin/當中
  - 至於本機自行安裝的軟體所產生的系統執行檔(system binary)，則放置到/usr/local/sbin/當中了
  - 常見的指令包括：fdisk, fsck, ifconfig, init, mkfs等等
- `/srv`: srv可以視為『service』的縮寫，是一些網路服務啟動之後，這些服務所需要取用的資料目錄。
  - 常見的服務例如WWW, FTP等等。舉例來說，WWW伺服器需要的網頁資料就可以放置在/srv/www/裡面。
- `/tmp`: 這是讓一般使用者或者是正在執行的程序暫時放置檔案的地方。 這個目錄是任何人都能夠存取的，所以需要定期清理一下。
  - 重要資料不可放置在此目錄！因為FHS甚至建議在開機時，應該要將/tmp下的資料都刪除


FHS針對根目錄所定義的標準就僅有上面，不過Linux底下還有許多其他重要目錄你需要瞭解一下：
- `/lost+found`: 這個目錄是使用標準的ext2/ext3檔案系統格式才會產生的一個目錄，目的在於當檔案系統發生錯誤時，將一些遺失的片段放置到這個目錄下
  - 這個目錄通常會在分割槽的最頂層存在， 例如你加裝一顆硬碟於/disk中，那在這個系統下就會自動產生一個這樣的目錄『/disk/lost+found』
- `/proc`: 這個目錄本身是一個『虛擬檔案系統(virtual filesystem)』！他放置的資料都是在記憶體當中，例如系統核心、行程資訊(process)、周邊裝置的狀態及網路狀態等等。
  - 因為這個目錄下的資料都是在記憶體當中， 所以本身不佔任何硬碟空間
  - 比較重要的檔案例如：/proc/cpuinfo, /proc/dma, /proc/interrupts, /proc/ioports, /proc/net/* 等等。
- `/sys`: 這個目錄其實跟/proc非常類似，也是一個虛擬的檔案系統，主要也是記錄與核心相關的資訊。 包括目前已載入的核心模組與核心偵測到的硬體裝置資訊等等。
  - 這個目錄同樣不佔硬碟容量

除了這些目錄的內容之外，另外要注意的是，因為根目錄與開機有關，開機過程中僅有根目錄會被掛載，其他分割槽則是在開機完成之後才會持續的進行掛載的行為。因為如此，因此根目錄下與開機過程有關的目錄， 就不能夠與根目錄放到不同的分割槽去！包含底下這些：
- /etc：設定檔
- /bin：重要執行檔
- /dev：所需要的裝置檔案
- /lib：執行檔所需的函式庫與核心所需的模組
- /sbin：重要的系統執行檔


#### /usr 的意義與內容
依據FHS的定義，/usr裡面放置的資料屬於可分享的與不可變動的(shareable, static)

很多人都會誤會/usr為user的縮寫，其實usr是Unix Software Resource的縮寫，也就是『Unix作業系統軟體資源』所放置的目錄。FHS建議所有軟體開發者，應該將他們的資料合理的分別放置到這個目錄下的次目錄，而不要自行建立該軟體自己獨立的目錄。

因為是所有系統預設的軟體(distribution發佈者提供的軟體)都會放置到/usr底下，因此這個目錄有點類似Windows 系統的『C:\Windows\ (當中的一部份) + C:\Program files\』這兩個目錄的綜合體，系統剛安裝完畢時，這個目錄會佔用最多的硬碟容量。一般來說，/usr的次目錄建議有底下這些：

- `/usr/X11R6/`: 為X Window System重要資料所放置的目錄，之所以取名為X11R6是因為最後的X版本為第11版，且該版的第6次釋出之意。
- `/usr/bin/`: 絕大部分的使用者可使用指令都放在這裡！請注意到他與/bin的不同之處。(是否與開機過程有關)
- `/usr/include/`: c/c++等程式語言的檔頭(header)與包含檔(include)放置處，當我們以tarball方式 (*.tar.gz 的方式安裝軟體)安裝某些資料時，會使用到裡頭的許多包含檔喔！
- `/usr/lib/`: 包含各應用軟體的函式庫、目標檔案(object file)，以及不被一般使用者慣用的執行檔或腳本(script)。
  - 某些軟體會提供一些特殊的指令來進行伺服器的設定，這些指令也不會經常被系統管理員操作，那就會被擺放到這個目錄下
  - 如果你使用的是X86_64的Linux系統， 那可能會有/usr/lib64/目錄產生
- `/usr/local/`: 系統管理員在本機自行安裝自己下載的軟體(非distribution預設提供者)，建議安裝到此目錄，這樣會比較便於管理。
  - 可以自行到/usr/local去看看，該目錄下也是具有bin, etc, include, lib...的次目錄喔！
- `/usr/sbin/`: 非系統正常運作所需要的系統指令。最常見的就是某些網路伺服器軟體的服務指令(daemon)囉
- `/usr/share/`: 放置共享文件的地方，在這個目錄下放置的資料幾乎是不分硬體架構均可讀取的資料，幾乎都是文字檔案！在此目錄下常見的還有這些次目錄：
  - `/usr/share/man`: 線上說明文件
  - `/usr/share/doc`: 軟體雜項的文件說明
  - `/usr/share/zoneinfo`: 與時區有關的時區檔案
  - `/usr/src/`: 一般原始碼建議放置到這裡，src有source的意思。至於核心原始碼則建議放置到/usr/src/linux/目錄下。

> **[/sbin, /bin, /usr/sbin, /usr/bin 差別](https://cloud.tencent.com/developer/article/1679890)**
> 這些目錄都是存放命令的，首先區別下/sbin和/bin：
> - 從命令功能來看，/sbin 下的命令屬於基本的系統命令，如shutdown，reboot，用於啟動系統，修復系統，/bin下存放一些普通的基本命令，如ls,chmod等，這些命令在Linux系統裡的配置文件腳本里經常用到。
> - 從用戶權限的角度看，/sbin目錄下的命令通常只有管理員才可以運行，/bin下的命令管理員和一般的用戶都可以使用。
> - 從可運行時間角度看，/sbin,/bin能夠在掛載其他文件系統前就可以使用。
> - 而/usr/bin,/usr/sbin與/sbin /bin目錄的區別在於
>   - /bin,/sbin目錄是在系統啟動後掛載到根文件系統中的，所以/sbin,/bin目錄必須和根文件系統在同一分區；
>   - /usr/bin,usr/sbin可以和根文件系統不在一個分區。
>   - /usr/sbin存放的一些非必須的系統命令；/usr/bin存放一些用戶命令，如led(控制LED燈的)。
> 
> - 轉一位網友的解讀：
>   - /bin是系統的一些指令。 bin為binary的簡寫主要放置一些系統的必備執行檔例如:cat、cp、chmod df、dmesg、gzip、kill、ls、mkdir、more、mount、rm、su、tar等
>   - /sbin一般是指超級用戶指令。主要放置一些系統管理的必備程式例如:cfdisk、dhcpcd、dump、e2fsck、fdisk、halt、ifconfig、ifup、 ifdown、init、insmod、lilo、lsmod、mke2fs、modprobe、quotacheck、reboot、rmmod、 runlevel、shutdown等
>   - /usr/bin　是你在後期安裝的一些軟件的運行腳本。主要放置一些應用軟體工具的必備執行檔例如c++、g++、gcc、chdrv、diff、dig、du、eject、elm、free、gnome*、 gzip、htpasswd、kfm、ktop、last、less、locale、m4、make、man、mcopy、ncftp、 newaliases、nslookup passwd、quota、smb*、wget等。


#### /var 的意義與內容
如果/usr是安裝時會佔用較大硬碟容量的目錄，那麼/var就是在系統運作後才會漸漸佔用硬碟容量的目錄。因為/var目錄主要針對常態性變動的檔案，包括快取(cache)、登錄檔(log file)以及某些軟體運作所產生的檔案，包括程序檔案(lock file, run file)，或者例如MySQL資料庫的檔案等等。常見的次目錄有：

- `/var/cache/`: 應用程式本身運作過程中會產生的一些暫存檔；
- `/var/lib/`: 程式本身執行的過程中，需要使用到的資料檔案放置的目錄。在此目錄下各自的軟體應該要有各自的目錄。
  - 例，MySQL的資料庫放置到/var/lib/mysql/, rpm的資料庫則放到/var/lib/rpm
- `/var/lock/`: 某些裝置或者是檔案資源一次只能被一個應用程式所使用，如果同時有兩個程式使用該裝置時，就可能產生一些錯誤的狀況，因此就得要將該裝置上鎖(lock)，以確保該裝置只會給單一軟體所使用。
  - 舉例來說，燒錄機正在燒錄一塊光碟，你想一下，會不會有兩個人同時在使用一個燒錄機燒片？
- `/var/log/`: 重要到不行！這是登錄檔放置的目錄！
  - 裡面比較重要的檔案如/var/log/messages, /var/log/wtmp(記錄登入者的資訊)等。
- `/var/mail/`: 放置個人電子郵件信箱的目錄，不過這個目錄也被放置到/var/spool/mail/目錄中！通常這兩個目錄是互為連結檔！
- `/var/run/`: 某些程式或者是服務啟動後，會將他們的PID放置在這個目錄下喔！ 至於PID的意義我們會在後續章節提到的。
- `/var/spool/`: 這個目錄通常放置一些佇列資料，所謂的『佇列』就是排隊等待其他程式使用的資料啦！這些資料被使用後通常都會被刪除
  - 系統收到新信會放置到/var/spool/mail/中，但使用者收下該信件後該封信原則上就會被刪除。
  - 信件如果暫時寄不出去會被放到/var/spool/mqueue/中，等到被送出後就被刪除
  - 如果是工作排程資料(crontab)，就會被放置到/var/spool/cron/目錄中



### 3.2 目錄樹(directory tree)

[directory tree](/images/linux_bootcamp/directory_tree.gif)

### 3.3 絕對路徑與相對路徑
### 3.4 CentOS 的觀察： lsb_release

某些時刻你可能想要知道你的 distribution 使用的是那個 Linux 標準 (Linux Standard Base)，而且我們也知道 distribution 使用的都是 Linux 的核心！那你如何觀察這些基本的資訊呢？ 可以使用如下的指令來觀察

```shell
[root@www ~]# uname -r
2.6.18-128.el5 <==可以察看實際的核心版本

[root@www ~]# lsb_release -a
LSB Version:    :core-3.1-amd64:core-3.1-ia32:core-3.1-noarch:graphics-3.1-amd64:
graphics-3.1-ia32:graphics-3.1-noarch      <==LSB 的版本
Distributor ID: CentOS
Description:    CentOS release 5.3 (Final) <==distribution 的版本
Release:        5.3
Codename:       Final
```
