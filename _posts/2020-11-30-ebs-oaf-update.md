---
layout: post
title: EBS 12.1 OAF input 欄位值放大
categories: EBS12
---

最近需要把 Oracle EBS 12.1 OAF 的某個 messageTextInput 從 2 放大為 3，記錄一下過程。因為是放大限制，所以不能透過 Personalize Page 設定，必須修改程式與相關設定。

## 更新 OAF Page 預設值

- 修改對應 view 的 XML (一般是在 webui 目錄下，不確定的話，可以從 app 網頁上找，或是從 application developer 的 functions 找)
- import 頁面到 EBS db:
{% highlight console %}
p9879989_R12_GENERIC\jdevbin\oaext\bin\import.bat <fullPath>\view.xml \
-rootdir <fullPath> -username <dbusername> -password <dbusername> \
-dbconnection \
"(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=)(PORT=)))(CONNECTION_DATA=(SID=)(SERVER=DEDICATED)))"
{% endhighlight %}
- 將 view.xml 複製到 middle tier 主機上的對應位置
- 登入 middle tier 主機
- cd $ADMIN_SCRIPT_HOME
- ./adoacorectl.sh stop
- ./adoacorectl.sh start
