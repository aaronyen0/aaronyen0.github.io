---
title: 鳥哥私房菜 ch6：檔案與目錄管理
date: 2022-11-27 00:00:00
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

# [鳥哥私房菜 ch6：檔案與目錄管理](https://linux.vbird.org/linux_basic/centos7/0220filemanager.php)
全文皆為幫助自己記憶所節錄的內容


## 6.1 目錄與路徑

### 6.1.1 相對路徑與絕對路徑

```shell
.         # 代表此層目錄
..        # 代表上一層目錄
-         # 代表前一個工作目錄
~         # 代表『目前使用者身份』所在的家目錄
~account  # 代表 account 這個使用者的家目錄(account是個帳號名稱)
```

### 6.1.2 目錄的相關操作： cd, pwd, mkdir, rmdir


```shell
# 變換目錄
cd

# 顯示目前的目錄
# -P  ：顯示出確實的路徑，而非使用連結 (link) 路徑。
pwd [-P]

# 建立一個新的目錄
# -m ：設定檔案的權限喔！直接設定，不需要看預設權限 (umask) 的臉色～
# -p ：幫助你直接將所需要的目錄(包含上層目錄)遞迴建立起來！
mkdir [-mp]
mkdir -p test1/test2/test3/test4
mkdir -m 711 test2

# 刪除一個空的目錄
# -p ：連同『上層』『空的』目錄也一起刪除
rmdir [-p]
rmdir -p test1/test2/test3/test4
rm -r test1  # 遞迴刪除，比較危險
```


### 6.1.3 關於執行檔路徑的變數： $PATH

ls完整檔名為：/bin/ls(這是絕對路徑)，為何只打ls能夠執行，因為有PATH的幫助
```shell
echo $PATH
# /usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
```

如果將ls由/bin/ls移動成為/root/ls，然後自己在/root目錄下， 請問
1. 能不能直接輸入ls來執行？
   1. 不能，/root不在PATH內
2. 若不能，該如何執行ls這個指令？
   1. `./ls` 相對路徑執行
   2. `/root/ls` 絕對路徑執行
3. 若要直接輸入ls即可執行，又該如何進行？
   1. `PATH="${PATH}:/root"` 將root加入PATH中

有時候不過系統還是會告知你無法處理 /root/ls！很可能是因為指令參數被快取的關係。只要登出 (exit) 再登入 (su -) 就可以使用ls了


而由上面的幾個例題我們也可以知道幾件事情：
- 不同身份使用者預設的PATH不同，預設能夠隨意執行的指令也不同(如root與dmtsai)；
- PATH是可以修改的；
- 使用絕對路徑或相對路徑直接指定某個指令的檔名來執行，會比搜尋PATH來的正確；
- 指令應該要放置到正確的目錄下，執行才會比較方便；
- 本目錄(.)最好不要放到PATH當中。


## 6.2 檔案與目錄管理
### 6.2.1 檔案與目錄的檢視： ls

```shell
[root@study ~]# ls [-aAdfFhilnrRSt] 檔名或目錄名稱..
[root@study ~]# ls [--color={never,auto,always}] 檔名或目錄名稱..
[root@study ~]# ls [--full-time] 檔名或目錄名稱..
選項與參數：
-a  ：全部的檔案，連同隱藏檔( 開頭為 . 的檔案) 一起列出來(常用)
-A  ：全部的檔案，連同隱藏檔，但不包括 . 與 .. 這兩個目錄
-d  ：僅列出目錄本身，而不是列出目錄內的檔案資料(常用)
-f  ：直接列出結果，而不進行排序 (ls 預設會以檔名排序！)
-F  ：根據檔案、目錄等資訊，給予附加資料結構，例如：*:代表可執行檔； /:代表目錄； =:代表 socket 檔案； |:代表 FIFO 檔案；
-h  ：將檔案容量以人類較易讀的方式(例如 GB, KB 等等)列出來；
-i  ：列出 inode 號碼，inode 的意義下一章將會介紹；
-l  ：長資料串列出，包含檔案的屬性與權限等等資料；(常用)
-n  ：列出 UID 與 GID 而非使用者與群組的名稱 (UID與GID會在帳號管理提到！)
-r  ：將排序結果反向輸出，例如：原本檔名由小到大，反向則為由大到小；
-R  ：連同子目錄內容一起列出來，等於該目錄下的所有檔案都會顯示出來；
-S  ：以檔案容量大小排序，而不是用檔名排序；
-t  ：依時間排序，而不是用檔名。
--color=never  ：不要依據檔案特性給予顏色顯示；
--color=always ：顯示顏色
--color=auto   ：讓系統自行依據設定來判斷是否給予顏色
--full-time    ：以完整時間模式 (包含年、月、日、時、分) 輸出
--time={atime,ctime} ：輸出 access 時間或改變權限屬性時間 (ctime), 而非內容變更時間 (modification time)
```

### 6.2.2 複製、刪除與移動： cp, rm, mv

#### cp
```shell
[root@study ~]# cp [-adfilprsu] 來源檔(source) 目標檔(destination)
[root@study ~]# cp [options] source1 source2 source3 .... directory
選項與參數：
-a  ：相當於 -dr --preserve=all 的意思，至於 dr 請參考下列說明；(常用)
-d  ：若來源檔為連結檔的屬性(link file)，則複製連結檔屬性而非檔案本身；
-f  ：為強制(force)的意思，若目標檔案已經存在且無法開啟，則移除後再嘗試一次；
-i  ：若目標檔(destination)已經存在時，在覆蓋時會先詢問動作的進行(常用)
-l  ：進行硬式連結(hard link)的連結檔建立，而非複製檔案本身；
-p  ：連同檔案的屬性(權限、用戶、時間)一起複製過去，而非使用預設屬性(備份常用)；
-r  ：遞迴持續複製，用於目錄的複製行為；(常用)
-s  ：複製成為符號連結檔 (symbolic link)，亦即『捷徑』檔案；
-u  ：destination 比 source 舊才更新 destination，或 destination 不存在的情況下才複製。
--preserve=all ：除了 -p 的權限相關參數外，還加入 SELinux 的屬性, links, xattr 等也複製了。
最後需要注意的，如果來源檔有兩個以上，則最後一個目的檔一定要是『目錄』才行！
```


在預設的條件中，cp 的來源檔與目的檔的權限是不同的，目的檔的擁有者通常會是指令操作者本身。當我們在進行備份的時候，某些需要特別注意的特殊權限檔案，例如密碼檔 (/etc/shadow) 以及一些設定檔，就不能直接以 cp 來複製，而必須要加上 -a 或者是 -p 等等可以完整複製檔案權限的選項才行！如果想複製檔案給其他的使用者，也必須要注意到檔案的權限(包含讀、寫、執行以及檔案擁有者等)，否則，其他人還是無法針對你給予的檔案進行修訂的動作

```shell
範例一：用root身份，將家目錄下的 .bashrc 複製到 /tmp 下，並更名為 bashrc
[root@study ~]# cp ~/.bashrc /tmp/bashrc
[root@study ~]# cp -i ~/.bashrc /tmp/bashrc  #加上-i會出現互動式詢問
cp: overwrite `/tmp/bashrc'? n  <==n不覆蓋，y為覆蓋

範例二：變換目錄到/tmp，並將/var/log/wtmp複製到/tmp且觀察屬性：
[root@study ~]# cd /tmp
[root@study tmp]# cp /var/log/wtmp . <==想要複製到目前的目錄，最後的 . 不要忘
[root@study tmp]# ls -l /var/log/wtmp wtmp
-rw-rw-r--. 1 root utmp 28416 Jun 11 18:56 /var/log/wtmp
-rw-r--r--. 1 root root 28416 Jun 11 19:01 wtmp
# 注意上面的特殊字體，在不加任何選項的情況下，檔案的某些屬性/權限/建立時間會改變

# 那如果你想要將檔案的所有特性都一起複製過來該怎辦？可以加上 -a
[root@study tmp]# cp -a /var/log/wtmp wtmp_2
[root@study tmp]# ls -l /var/log/wtmp wtmp_2
-rw-rw-r--. 1 root utmp 28416 Jun 11 18:56 /var/log/wtmp
-rw-rw-r--. 1 root utmp 28416 Jun 11 18:56 wtmp_2
# 整個資料特性完全一模一樣！
```

```shell
範例三：複製 /etc/ 這個目錄下的所有內容到 /tmp 底下
[root@study tmp]# cp -r /etc/ /tmp
# 還是要再次強調！ -r 是可以複製目錄，但是，檔案與目錄的權限可能會被改變
# 所以，也可以利用『 cp -a /etc /tmp 』來下達指令！尤其是在備份的情況下

範例四：將範例一複製的 bashrc 建立一個連結檔 (symbolic link)
[root@study tmp]# ls -l bashrc
-rw-r--r--. 1 root root 176 Jun 11 19:01 bashrc  <==先觀察一下檔案情況

[root@study tmp]# cp -s bashrc bashrc_slink
[root@study tmp]# cp -l bashrc bashrc_hlink
[root@study tmp]# ls -l bashrc*
-rw-r--r--. 2 root root 176 Jun 11 19:01 bashrc                   <==原始檔案，link數變成2
-rw-r--r--. 2 root root 176 Jun 11 19:01 bashrc_hlink             <==連結到同一個inode，所以link數也是2
lrwxrwxrwx. 1 root root   6 Jun 11 19:06 bashrc_slink -> bashrc   <==捷徑，目標被刪除會不能開
```

```shell
範例五：若 ~/.bashrc 比 /tmp/bashrc 新才複製過來
[root@study tmp]# cp -u ~/.bashrc /tmp/bashrc
# 這個 -u 的特性，是在目標檔案與來源檔案有差異時，才會複製的。所以，比較常被用於『備份』工作

範例六：將範例四造成的 bashrc_slink 複製成為 bashrc_slink_1 與bashrc_slink_2
[root@study tmp]# cp bashrc_slink bashrc_slink_1        <==沒加參數，即使複製的是連結檔，仍會將實際檔案複製
[root@study tmp]# cp -d bashrc_slink bashrc_slink_2     <==依然保持連結檔的特性
[root@study tmp]# ls -l bashrc bashrc_slink*
-rw-r--r--. 2 root root 176 Jun 11 19:01 bashrc
lrwxrwxrwx. 1 root root   6 Jun 11 19:06 bashrc_slink -> bashrc
-rw-r--r--. 1 root root 176 Jun 11 19:09 bashrc_slink_1            <==與原始檔案相同
lrwxrwxrwx. 1 root root   6 Jun 11 19:10 bashrc_slink_2 -> bashrc  <==是連結檔！
# bashrc_slink_1原本複製的是連結檔，但是卻將連結檔的實際檔案複製過來了
# bashrc_slink_2要複製連結檔的屬性，使用 -d 的選項

範例七：將家目錄的 .bashrc 及 .bash_history 複製到 /tmp 底下
[root@study tmp]# cp ~/.bashrc ~/.bash_history /tmp
# 可以將多個資料一次複製到同一個目錄去！最後面一定是目錄！
```


#### rm

```shell
[root@study ~]# rm [-fir] 檔案或目錄
選項與參數：
-f  ：就是 force 的意思，忽略不存在的檔案，不會出現警告訊息；
-i  ：互動模式，在刪除前會詢問使用者是否動作
-r  ：遞迴刪除啊！最常用在目錄的刪除了！這是非常危險的選項！！！

範例一：將剛剛在 cp 的範例中建立的 bashrc 刪除掉！
[root@study ~]# cd /tmp
[root@study tmp]# rm -i bashrc
rm: remove regular file `bashrc'? y
# 如果加上 -i 的選項就會主動詢問，避免你刪除到錯誤的檔名！

範例三：將 cp 範例中所建立的 /tmp/etc/ 這個目錄刪除掉！
[root@study tmp]# rmdir /tmp/etc
rmdir: failed to remove '/tmp/etc': Directory not empty   <== 刪不掉啊！因為這不是空的目錄！
[root@study tmp]# rm -r /tmp/etc
rm: descend into directory `/tmp/etc'? y
.....(中間省略).....
# 因為身份是 root ，預設已經加入了 -i 的選項，所以你要一直按 y 才會刪除！
# 這是一種保護的動作，如果確定要刪除掉此目錄而不要詢問，可以在指令前加上反斜線：
[root@study tmp]# \rm -r /tmp/etc
# 在指令前加上反斜線，可以忽略掉 alias 的指定選項喔！至於 alias 我們在bash再談！
# 這個範例很可怕！錯刪 /etc 系統會掛掉
```


### mv

```shell
[root@study ~]# mv [-fiu] source destination
[root@study ~]# mv [options] source1 source2 source3 .... directory
選項與參數：
-f  ：force 強制的意思，如果目標檔案已經存在，不會詢問而直接覆蓋；
-i  ：若目標檔案 (destination) 已經存在時，就會詢問是否覆蓋！
-u  ：若目標檔案已經存在，且 source 比較新，才會更新 (update)
```


### 6.2.3 取得路徑的檔案名稱與目錄名稱

```shell
[root@study ~]# basename /etc/sysconfig/network
network         <== 取得最後的檔名～
[root@study ~]# dirname /etc/sysconfig/network
/etc/sysconfig  <== 取得目錄名了！
```

## 6.3 檔案內容查閱

如果我們要查閱一個檔案的內容時，有相當多有趣的指令：最常使用的顯示檔案內容的指是cat與more及less！
此外，如果我們要查看一個很大型的檔案 (好幾百MB時)，但是我們只需要後端的幾行字而已，可以用 tail或是tac

- cat  由第一行開始顯示檔案內容
- tac  從最後一行開始顯示，可以看出 tac 是 cat 的倒著寫！
- nl   顯示的時候，順道輸出行號！
- more 一頁一頁的顯示檔案內容
- less 與 more 類似，但是比 more 更好的是，他可以往前翻頁！
- head 只看頭幾行
- tail 只看尾巴幾行
- od   以二進位的方式讀取檔案內容！

### 6.3.1 直接檢視檔案內容： cat, tac, nl


```shell
# cat (concatenate)
[root@study ~]# cat [-AbEnTv]
選項與參數：
-A  ：相當於 -vET 的整合選項，可列出一些特殊字符而不是空白而已；
-b  ：列出行號，僅針對非空白行做行號顯示，空白行不標行號！
-E  ：將結尾的斷行字元 $ 顯示出來；
-n  ：列印出行號，連同空白行也會有行號，與 -b 的選項不同；
-T  ：將 [tab] 按鍵以 ^I 顯示出來；
-v  ：列出一些看不出來的特殊字符
```


```shell
# tac(反向列示)，tac 剛好是將 cat 反寫過來
[root@study ~]# tac 
```



```shell
# nl (添加行號列印)
[root@study ~]# nl [-bnw] 檔案
選項與參數：
-b  ：指定行號指定的方式，主要有兩種：
      -b a ：表示不論是否為空行，也同樣列出行號(類似 cat -n)；
      -b t ：如果有空行，空的那一行不要列出行號(預設值)；
-n  ：列出行號表示的方法，主要有三種：
      -n ln ：行號在螢幕的最左方顯示；
      -n rn ：行號在自己欄位的最右方顯示，且不加 0 ；
      -n rz ：行號在自己欄位的最右方顯示，且加 0 ；
-w  ：行號欄位的佔用的字元數。

範例一：用 nl 列出 /etc/issue 的內容
[root@study ~]# nl /etc/issue
     1  \S
     2  Kernel \r on an \m

# 注意看，這個檔案其實有三行，第三行為空白(沒有任何字元)，
# 因為他是空白行，所以 nl 不會加上行號喔！如果確定要加上行號，可以這樣做：

[root@study ~]# nl -b a /etc/issue
     1  \S
     2  Kernel \r on an \m
     3
# 呵呵！行號加上來囉～那麼如果要讓行號前面自動補上 0 呢？可這樣

[root@study ~]# nl -b a -n rz /etc/issue
000001  \S
000002  Kernel \r on an \m
000003
# 嘿嘿！自動在自己欄位的地方補上 0 了～預設欄位是六位數，如果想要改成 3 位數？

[root@study ~]# nl -b a -n rz -w 3 /etc/issue
001     \S
002     Kernel \r on an \m
003
# 變成僅有 3 位數囉～
```

### 6.3.2 可翻頁檢視： more, less
#### less (一頁一頁翻動)
less 的用法比起 more 更加的有彈性，在 more 的時候，我們並沒有辦法向前面翻， 只能往後面看，但若使用了 less 時，就可以使用 pageup, pagedown 等按鍵的功能來往前往後翻看文件，在 less 裡頭可以擁有更多的『搜尋』功能：

- 空白鍵    ：向下翻動一頁；
- [pagedown]：向下翻動一頁；
- [pageup]  ：向上翻動一頁；
- /字串     ：向下搜尋『字串』的功能；
- ?字串     ：向上搜尋『字串』的功能；
- n         ：重複前一個搜尋 (與 / 或 ? 有關！)
- N         ：反向的重複前一個搜尋 (與 / 或 ? 有關！)
- g         ：前進到這個資料的第一行去；
- G         ：前進到這個資料的最後一行去 (注意大小寫)；
- q         ：離開 less 這個程式；


### 6.3.3 資料擷取： head, tail

```shell
[root@study ~]# head [-n number] 檔案 
選項與參數：
-n  ：後面接數字，代表顯示幾行的意思

[root@study ~]# head /etc/man_db.conf
# 預設的情況中，顯示前面十行！若要顯示前 20 行，就得要這樣：
[root@study ~]# head -n 20 /etc/man_db.conf

範例：如果後面100行的資料都不列印，只列印/etc/man_db.conf的前面幾行，該如何是好？
[root@study ~]# head -n -100 /etc/man_db.conf
```

```shell
[root@study ~]# tail [-n number] 檔案 
選項與參數：
-n  ：後面接數字，代表顯示幾行的意思
-f  ：表示持續偵測後面所接的檔名，要等到按下[ctrl]-c才會結束tail的偵測

[root@study ~]# tail /etc/man_db.conf
# 預設的情況中，顯示最後的十行！若要顯示最後的 20 行，就得要這樣：
[root@study ~]# tail -n 20 /etc/man_db.conf

範例一：如果不知道/etc/man_db.conf有幾行，卻只想列出100行以後的資料時？
[root@study ~]# tail -n +100 /etc/man_db.conf

範例二：持續偵測/var/log/messages的內容
[root@study ~]# tail -f /var/log/messages
  <==要等到輸入[ctrl]-c之後才會離開tail這個指令的偵測！
```

> 假如我想要顯示 /etc/man_db.conf 的第 11 到第 20 行呢？
> 在第 11 到第 20 行，那麼我取前 20 行，再取後十行，所以結果就是：`head -n 20 /etc/man_db.conf | tail -n 10`

> 如果我想要列出正確的行號呢？就是螢幕上僅列出 /etc/man_db.conf 的第 11 到第 20 行，且有行號存在？
> 可以透過 cat -n 來帶出行號，然後再透過 head/tail 來擷取資料`cat -n /etc/man_db.conf | head -n 20 | tail -n 10`

### 6.3.4 非純文字檔： od
### 6.3.5 修改檔案時間與建置新檔： touch
## 6.4 檔案與目錄的預設權限與隱藏權限
### 6.4.1 檔案預設權限：umask
### 6.4.2 檔案隱藏屬性： chattr, lsattr
### 6.4.3 檔案特殊權限：SUID, SGID, SBIT, 權限設定
### 6.4.4 觀察檔案類型：file
## 6.5 指令與檔案的搜尋
### 指令檔名的搜尋：which
### 檔案檔名的搜尋：whereis, locate / updatedb, find
## 6.6 極重要的複習！權限與指令間的關係
## 6.7 重點回顧
## 6.8 本章習題
## 6.9 參考資料與延伸閱讀
## 針對本文的建議：http://phorum.vbird.org/viewtopic.php?t=23879