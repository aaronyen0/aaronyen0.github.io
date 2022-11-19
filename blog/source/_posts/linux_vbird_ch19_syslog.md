---
title: 鳥哥私房菜 ch19：syslog
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

影片過於粗略，改看[鳥哥私房菜19章 認識分析登錄檔](https://linux.vbird.org/linux_basic/centos5/0570syslog-centos5.php)


以下內容皆為節錄鳥哥私房菜，無任何營利用途，單純一邊閱讀一邊節錄重點幫助記憶


# 認識分析登錄檔

## 什麼是登錄檔
### 登錄檔的重要性
- 記錄系統活動資訊的幾個檔案，記錄系統在什麼時候由哪個程序做了什麼樣的行為時，發生了何種的事件等等。
- 重要性
  - 解決系統方面的錯誤：偶爾會發現系統可能會出現一些錯誤，包括硬體捉不到或者是某些系統程式無法順利運作的情況。由於系統會將硬體偵測過程記錄在登錄檔內，你只要透過查詢登錄檔就能夠瞭解系統作了啥
- 解決網路服務的問題
  - 你可能在做完了某些網路服務的設定後，卻一直無法順利啟動該服務，此時該怎辦?
    - 由於網路服務的各種問題通常都會被寫入特別的登錄檔，其實只要查詢登錄檔就會知道出了什麼差錯
    - 舉例來說，如果你無法啟動郵件伺服器(sendmail)，那麼查詢一下 /var/log/maillog 通常可以得到不錯的解答
- 過往事件記錄簿
  - 例如你發現WWW服務(apache 軟體)在某個時刻流量特別大，你想要瞭解為什麼，可以透過登錄檔去找出該時段是哪些IP在連線與查詢的網頁資料為何。 
  - 此外，萬一系統被入侵，並且被利用來攻擊他人的主機，由於被攻擊主機會記錄攻擊者，因此你的 IP 就會被對方記錄。這個時候系統登入檔可以告知對方你的主機是由於被入侵所導致的問題，並且協助對方追查惡意來源

### 常見檔名
登錄檔的權限通常是設定為僅有 root 能夠讀取而已
- /var/log/cron：
  - crontab排程有沒有實際被進行？
  - 進行過程有沒有發生錯誤？
  - /etc/crontab 是否撰寫正確？

- /var/log/dmesg：
  - 記錄系統在開機的時候核心偵測過程所產生的各項資訊

- /var/log/lastlog：
  - 記錄系統上面所有的帳號最近一次登入系統時的相關資訊

- /var/log/maillog 或 /var/log/mail/*
  - 記錄郵件的往來資訊
  - 主要是記錄 sendmail (SMTP協定提供者) 與 dovecot (POP3協定提供者)所產生的訊息 
  - SMTP 是發信所使用的通訊協定， POP3 則是收信使用的通訊協定 
  - sendmail 與 dovecot 則分別是兩套達成通訊協定的軟體

- /var/log/messages
  - 幾乎系統發生的錯誤訊息 (或者是重要的資訊) 都會記錄在這個檔案中
  - 如果系統發生莫名的錯誤時，這個檔案是一定要查閱的登錄檔之一

- /var/log/secure
  - 基本上，只要牽涉到「需要輸入帳號密碼」的軟體
  - 那麼當登入時 (不管登入正確或錯誤) 都會被記錄在此檔案中
  - 包括系統的 login 程式、圖形介面登入所使用的 gdm 程式、su, sudo 等程式、還有網路連線的 ssh, telnet 等程式，登入資訊都會被記載在這裡

- /var/log/wtmp, /var/log/faillog
  - 這兩個檔案可以記錄正確登入系統者的帳號資訊(wtmp)
  - 錯誤登入時所使用的帳號資訊 (faillog)
  - 對於追蹤一般帳號者的使用行為很有幫助！

- /var/log/httpd/*, /var/log/news/*, /var/log/samba/*
  - 不同的網路服務會使用它們自己的登錄檔案來記載它們自己產生的各項訊息、上述目錄內則是個別服務所制訂的登錄檔。

### 服務(daemon)與程式
- 登錄檔是怎麼產生的
  - 由軟體開發商自行定義寫入的登錄檔與相關格式，例如WWW軟體apache就是這樣處理的
  - 另一種則是由 Linux distribution 提供的登錄檔管理服務來統一管理，只要將訊息丟給這個服務後，他就會自己分門別類的將各種訊息放置到相關的登錄檔去。
- 針對登錄檔所需的功能，我們需要的服務與程式有
  - syslogd：主要登錄系統與網路等服務的訊息
  - klogd：主要登錄核心產生的各項資訊
  - logrotate：主要在進行登錄檔的輪替功能

## syslogd：記錄登錄檔的服務
```shell
[root@www ~]# ps aux | grep syslog
USER   PID %CPU %MEM  VSZ  RSS TTY  STAT START  TIME COMMAND
root  4294  0.0  0.0 1716  568 ?    Ss   Mar31  0:00 syslogd -m 0
# 確實有啟動syslogd的！

[root@www ~]# chkconfig --list syslog
syslog    0:off  1:off  2:on   3:on   4:on   5:on   6:off
# 預設情況下，文字介面與圖形介面 3-5 都有啟動喔
```

### 登錄檔內容的一般格式
- syslog 而記錄下來的資料中，每條訊息均會記錄底下的幾個重要資料
  - 事件發生的日期與時間
  - 發生此事件的主機名稱
  - 啟動此事件的服務名稱 (如 samba, xinetd 等) 或函式名稱 (如 libpam ..)
  - 該訊息的實際資料內容

```shell
[root@www ~]# cat /var/log/secure
1 Mar 14 15:38:00 www atd[18701]: pam_unix(atd:session): session opened for user root by (uid=0)
# 在三月14日 (Mar 14) 的下午 15:38 分，由 www 這部主機的 atd [PID 為 18701] 傳來的消息，這個消息是透過 pam_unix 這個模組所提出的。訊息內容為 root (uid=0) 這個帳號已經開啟 atd 的活動了。

2 Mar 14 15:38:00 www atd[18701]: pam_unix(atd:session): session closed for user root
3 Mar 16 16:01:51 www su: pam_unix(su-l:auth): authentication failure; logn ame=vbird uid=500 euid=0 tty=pts/1 ruser=vbird rhost=  user=root
4 Mar 16 16:01:55 www su: pam_unix(su-l:session): session opened for user root by vbird(uid=500)
5 Mar 16 16:02:22 www su: pam_unix(su-l:session): session closed for user root

|--日期/時間---|-H-|-----服務與相關函數-------|--訊息說明------>
```

- 還有很多的資訊值得查閱！像是/var/log/messages的內容。好的系統管理員，要常常巡視登錄檔的內容，尤其是發生底下幾種情況時
  - 當你覺得系統不太正常
  - 某個 daemon 無法正常啟動時
  - 某個使用者無法登入時
  - 某個 daemon 執行過程不順暢時


- Tips: 當鳥哥無法成功的啟動服務時，會在最後一次啟動該服務後
  - 找到現在時間所登錄的資訊「第一欄位」
  - 找到我想要查詢的那個服務「第三欄位」
  - 再仔細的查閱第四欄位的資訊，來藉以找到錯誤點


### syslogd的設定檔
- syslog 針對各種服務與訊息記錄在某些檔案的設定檔就是 /etc/syslog.conf
  - 什麼服務
  - 的什麼等級訊息
  - 需要被記錄在哪裡(裝置或檔案)

```shell
#格式: <service-name>[.=!]<message-level> <filename or host>

mail.info /var/log/maillog_info
# mail 服務產生的大於等於 info 等級的訊息，都記錄到/var/log/maillog_info
```
#### 服務名稱: syslog 本身有規範一些服務，你可以透過這些服務來儲存系統的訊息，可用 `man 3 syslog` 查詢到相關資訊
|服務類別|說明|
|----|----|
| auth (authpriv) | 與認證有關的機制，例如 login, ssh, su 等與帳號/密碼有關的紀錄 |
| cron | 就是例行性工作排程 cron/at 等產生訊息記錄的地方 |
| daemon | 與各個 daemon 有關的訊息 |
| kern | 就是核心 (kernel) 產生訊息的地方 |
| lpr | 亦即是列印相關的訊息 |
| mail | 只要與郵件收發有關的訊息紀錄 |
| news | 與新聞群組伺服器有關 |
| syslog | 就是 syslogd 這支程式本身產生的資訊 |
| user, uucp, local0 ~ local7 | 與 Unix like 機器本身有關的一些訊息 |

#### 訊息等級
| 等級 | 等級名稱 | 說明 |
|-----|-----|-----|
| 1   | info | 基本的訊息說明而已 |
| 2   | notice | 比 info 還需要被注意到的一些資訊內容 |
| 3   | warning (warn) | 警示的訊息，可能有問題，但是還不至於影響到某個 daemon 運作的資訊 |
| 4   | err (error) | 一些重大的錯誤訊息，例如設定檔的某些設定值造成該服務服法啟動的資訊說明 |
| 5   | crit | 比 error 還要嚴重的錯誤資訊 |
| 6   | alert | 警告，已經很有問題的等級 |
| 7   | emerg (panic) | 意指系統已經幾乎要當機的狀態，通常只有硬體出問題，導致整個核心無法順利運作，才會出現這樣的等級的訊息 |

- 還有兩個特殊的等級，那就是debug與none(不需登錄等級)，當我們想要作錯誤偵測，或者忽略掉某些服務的資訊時，可以用這兩個
- 訊息等級前有`[.=!]`連結符號，意義為
```shell
. # 代表比後面還要高的等級 (含該等級) 都被記錄下來
.= # 代表所需要的等級就是後面接的等級而已，其他的不要
.! # 代表不等於，亦即是除了該等級外的其他等級都記錄
```

#### 紀錄的檔名或裝置或主機
- 檔案的絕對路徑：通常就是放在 /var/log 裡頭的檔案
- 印表機或其他：例如 /dev/lp0 這個印表機裝置
- 使用者名稱：顯示給使用者
- 遠端主機：例如 @www.vbird.tsai 同時要對方主機也能支援才行
- *：目前在線上的所有人，類似wall指令


#### 自行增加登錄檔檔案功能
- 直接加在/etc/syslog.conf上即可

```shell
# ex: 所有的資訊都額外寫入到 /var/log/admin.log 這個檔案

[root@www ~]# vim /etc/syslog.conf
*.info      /var/log/admin.log

# 重新啟動 syslog
[root@www ~]# /etc/init.d/syslog restart
[root@www ~]# ll /var/log/admin.log
-rw------- 1 root root 118 Apr  8 13:50 /var/log/admin.log
```

### 登錄檔的安全性設置
- 駭客為了湮滅證據，可能在離開的時候將所有登錄檔清除，例如/var/log整個目錄不見
  - 透過lsattr, chattr鎖定登錄檔的一些編輯行為，小心使用
  - 但是也會導致logrotate無法正常輪替，雖然也能透過設定來解決
- Command
  - `chattr +a /var/log/messages`: 檔案資料只能新增不能刪除
  - `chattr -a /var/log/messages`: 取消+a的flag

### 登錄檔伺服器的設定
想像辦公室內有十部主機，每一部負責一個網路服務，為了要瞭解每部主機的狀態，常常需要登入這十部主機去查閱你的登錄檔。這個時候我們可以讓某一部主機當成「登錄檔伺服器」，用他來記錄所有的十部主機的資訊

- syslog本身有登錄檔伺服器的功能了，只是預設並沒有啟動該功能，登錄檔伺服器預設的埠口是UDP的514

![syslog_server]](/images/linux_bootcamp/syslog_server.gif)

```shell
# 1. Server 端：修改 syslogd 的啟動設定檔，通常在 /etc/sysconfig內

[root@www ~]# vim /etc/sysconfig/syslog
# 找到底下這一行：
SYSLOGD_OPTIONS="-m 0"
# 改成
SYSLOGD_OPTIONS="-m 0 -r"

# 2. 重新啟動與觀察 syslogd
[root@www ~]# /etc/init.d/syslog restart
[root@www ~]# netstat -lunp | grep syslog
Proto Recv-Q Send-Q Local Address  Foreign Address State   PID/Program name
udp        0      0 0.0.0.0:514    0.0.0.0:*               13981/syslogd
# 你的登錄檔主機已經設定妥當
```

```shell
# Client端

[root@www ~]# vim /etc/syslog.conf
*.*       @192.168.1.100
```

## 登錄檔的輪替 (logrotate)
syslog 利用的是 daemon 的方式來啟動的，當有需求的時候立刻就會被執行的，但是logrotate是在規定的時間到了之後才來進行登錄檔的輪替，所以這個 logrotate是掛在 cron底下進行的
- /etc/cron.daily/logrotate

### logrotate 的設定檔
- 參數
  - `/etc/logrotate.conf`: 主要的參數檔案
  - `/etc/logrotate.d/`: 一個目錄，裡面所有檔案都會被主動的讀入`/etc/logrotate.conf`

- `/etc/logrotate.conf`
```shell
[root@www ~]# vim /etc/logrotate.conf
# 底下的設定是 "logrotate 的預設設定值"
# 如果個別的檔案設定了其他的參數，則將以個別的檔案設定為主

weekly    <==預設每個禮拜對登錄檔進行一次 rotate 的工作
rotate 4  <==保留幾個登錄檔呢？預設是保留四個！
create    <==由於登錄檔被更名，因此建立一個新的來繼續儲存之意！
#compress <==被更動的登錄檔是否需要壓縮？如果登錄檔太大則可考慮此參數啟動

include /etc/logrotate.d
# 將 /etc/logrotate.d/ 這個目錄中的所有檔案都讀進來執行 rotate 的工作！

/var/log/wtmp {       <==僅針對 /var/log/wtmp 所設定的參數
    monthly           <==每個月一次，取代每週！
    minsize 1M        <==檔案容量一定要超過1M才進行rota (略過時間參數)
    create 0664 root utmp <==指定新建檔案的權限與所屬帳號/群組
    rotate 1          <==僅保留一個，亦即僅有 wtmp.1 保留而已。
}
# 這個 wtmp 可記錄登入者與系統重新開機時的時間與來源主機及登入期間的時間。
# 由於具有 minsize 的參數，因此不見得每個月一定會進行一次喔！要看檔案容量。
# 由於僅保留一個登錄檔而已，不滿意的話可以將他改成 rotate 5 吧！
```

- `/etc/logrotate.d/syslog`
```shell
[root@www ~]# vi /etc/logrotate.d/syslog
/var/log/messages /var/log/secure /var/log/maillog /var/log/spooler \
/var/log/boot.log /var/log/cron {
  sharedscripts
  prerotate
    /usr/bin/chattr -a /var/log/messages
  endscript
  sharedscripts
  postrotate
    /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
    /bin/kill -HUP `cat /var/run/rsyslogd.pid 2> /dev/null` 2> /dev/null || true
    /usr/bin/chattr +a /var/log/messages
  endscript
}

# /bin/kill -HUP ... 目的在於將系統的 syslogd 重新以其參數檔 (syslog.conf) 的資料讀入一次
# 由於我們建立了一個新的空的紀錄檔，如果不執行此一行來重新啟動服務的話， 那麼記錄的時候將會發生錯誤
```

- 檔名：被處理的登錄檔絕對路徑檔名寫在前面，可以使用空白字元分隔多個登錄檔；
- 參數：上述檔名進行輪替的參數使用 { } 包括起來；
- 執行腳本：可呼叫外部指令來進行額外的命令下達，這個設定需與sharedscripts .... endscript合用
  - prerotate：在啟動 logrotate 之前進行的指令，例如修改登錄檔的屬性等動作；
  - postrotate：在做完 logrotate 之後啟動的指令，例如重新啟動 (kill -HUP) 某個服務！

### 自訂登錄檔的輪替功能


## 分析登錄檔
### CentOS 預設提供的 logwatch
- http://www.logwatch.org/


### 鳥哥自己寫的登錄檔分析工具：
- http://linux.vbird.org/download/index.php?action=detail&fileid=69



## Command
``` shell
# generate syslog: used for testing config change 
logger [options] message
# -p FACILITY.SERVERITY
# -t TAG
```