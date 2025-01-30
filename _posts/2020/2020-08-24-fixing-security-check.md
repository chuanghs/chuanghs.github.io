---
layout: post
title: OpenVAS 弱點掃描漏洞修正
categories: system admin
---

公司定期弱點掃描（[OpenVAS](https://www.openvas.org)），有台機器被掃到幾個還沒補上的洞，記錄一下

# Apache 安全性

## 舊版 SSL 協定未關閉
- 20007 - SSL Version 2 and 3 Protocol Detection
- 104743 - TLS Version 1.0 Protocol Detection
- 65821 - SSL RC4 Cipher Suites Supported (Bar Mitzvah)

這幾個問題比較簡單，主要就是一些已經停用的舊加密方式沒有關閉，容易被利用，修改 apache 的相關設定就行了：

### 修正方式
1. 編輯 `/etc/httpd/conf.d/ssl.conf`
2. 將 `SSLProtocol` 修改為 `SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1`
3. 將 `SSLCipherSuite` 改為 `SSLCipherSuite HIGH:3DES:!aNULL:!MD5:!SEED:!IDEA`
4. 重新啟動 httpd (`systemctl restart httpd`)

### 驗證

執行 `nmap --script ssl-enum-ciphers -p <port> <hostname>`，會列出主機目前支援的協定與 cipher，修改後應該只剩下 TLSv1.2，而且使用的 cipher 不會有 RC4，這台主機修改後的結果是：


```console
Host is up (0.0011s latency).

PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers: 
|   TLSv1.2: 
|     ciphers: 
|       TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA (dh 2048) - C

```



## 11213 - HTTP TRACE / TRACK Methods Allowed

這個問題有兩種解法，修改 httpd.conf 或用 Rewrite

### 修改 httpd.conf
編輯 `/etc/httpd/conf/httpd.conf`，在最後加上 `TraceEnable off`

### Rewrite


```apache
 RewriteEngine On
 RewriteCond %{REQUEST_METHOD} ^(TRACE|TRACK)
 RewriteRule .* - [F]
```

修改後都需要重新啟動 httpd（`systemctl restart httpd`）

### 驗證
執行 `./test4trac.pl <hostname>`，執行結果：

```console
First we test for Trace...
405 Method Not Allowed
TRACE is not permitted.

Now we test for Track...
501 Not Implemented
TRACK is not implemented.

```

驗證工具 [Test4Trac](https://blog.techstacks.com/test4trac.html)

# SSH 安全性

## 90317 - SSH Weak Algorithms Supported

### 修正方式

編輯 `/etc/ssh/sshd_config` 設定檔，加入：
`Ciphers 3des-cbc,cast128-cbc,aes128-cbc,aes192-cbc,aes256-cbc,rijndael-cbc@lysator.liu.se,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com`
就行了。

比較好的方式應該是用負面表列的方式，但目前這台主機的 ssh 還不支援負面表列。

修改設定檔後重新啟動 (systemctl restart sshd)或重新載入設定檔（systemctl reload sshd）。

### 驗證方式

執行 `ssh hostname -c arcfour` 應該會回應


```console
Unable to negotiate with ::1 port 22: no matching cipher found. Their offer: 3de
s-cbc,cast128-cbc,aes128-cbc,aes192-cbc,aes256-cbc,rijndael-cbc@lysator.liu.se,
aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com,
chacha20-poly1305@openssh.com
```
