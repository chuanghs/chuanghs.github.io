---
layout: post
title: 從 Ubuntu 22.04 轉移到 Debian testing (bookworm)
categories: [Linux, Ubuntu, Debian]
---

由於 Ubuntu 轉用 Snaps 作為套件管理方式，決定把手邊最後一台 Ubuntu 22.04 轉換到 Debian，由於是自用的 server，自然是轉換到
testing 版本囉，目前 Debian 的 testing 版本的 code name 是 bookworm. 

轉換步驟

1. 在 `/etc/apt/sources.list` 中加入 Debian 相關的 source
   ```
   deb http://deb.debian.org/debian bookworm main contrib non-free
   deb http://deb.debian.org/debian-security/ bookworm-security main contrib non-free 
   deb http://deb.debian.org/debian bookworm-updates main contrib non-free
   ```
2. 接著停用 Ubuntu 相關的套件來源，建立 `/etc/apt/preferences.d/10-no-ubuntu` 檔案，內容如下：
   ```
   Package: *
   Pin: release o=Ubuntu
   Pin-Priority: -1000
   ```
3. 手動安裝 Debian 的 keyring 檔案：搜尋 `debian-keyring` 與 `debian-archive-keyring` 檔案，下載後，再用 `dpkg -i debian*.deb` 安裝
4. 接下來就可以執行 `apt-get update` 了
5. 然後執行 `apt-get dist-upgrade`，若是 Ubuntu 使用的版本較新，應該會被降版才對。
   - 過程中可能會發生因為 ubuntu 套件無法移除卡住的狀況，可以用 `dpkg --purge --force-all <package>` 的方式強制移除 Ubuntu 套件
6. 接著移除 `/etc/apt/sources.list` 中的 Ubuntu 來源，並刪除 `/etc/apt/preferences.d/10-no-ubuntu` 檔案。
7. 檢查是否有 `linux-image-amd64` 之類名稱的套件，Ubuntu 的 kernel 套件使用不同的名稱