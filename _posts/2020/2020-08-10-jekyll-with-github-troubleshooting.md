---
layout: post
title: Jekyll + GitHub Page 問題排除
categories: jekyll
---

總算搞定 Jekyll + Github Page 了，遇到了以下問題

# 以 /blog 建立 repository

一開始想都沒想就用 /blog 建立 repository，結果是首頁可以成功出現，但一點進個別的文章，就會出現 404 錯誤，打開原始碼一看，發現 `href` 值漏了 `/blog` 這層。

![404 Page not Found](/assets/github-page-not-found.png)

解法是修改 _config.yml 指定 `baseurl` 值就行了，最後設定如下：

`
baseurl: "/blog"
`

# 特殊名稱的 repository

在解決前一個問題過程中，發現有人是用 `<username>.github.io` 的方式建立，測試之後發現，只要用 Domain name 的方式建立的 repository，就會把 GitHub Page 掛成只有 domain naem 的型式，不需要 context path，不用指定 `baseurl`，文章連結就可以正常運作（根本沒有 context path 怎麼會有問題）。
