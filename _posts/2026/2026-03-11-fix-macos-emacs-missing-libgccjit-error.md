---
layout: post
title: 修正 emacs 在 macOS 下找不到 libgccjit.so 導致編譯失敗
categories: [ 'emacs', 'macOS', 'fix' ]
tags: [ 'emacs', 'macOS', 'fix' ]
---

最近更新完 homebrew package 後，每次打開 emacs 都會有 ```libgccjit.dylib: No such file or directory``` 的錯誤，google 了一大圈，
都是前先版本的修法，只好依著自己的環境作了些調整，最後統整為以下三個步驟，以 homebrew 作為套作管理工具。

1. 確認正確安裝 gcc
  ```bash
   brew unlink gcc
   brew install gcc
  ```
1. 在 ```.zshrc``` 作以下修改
   ```bash
   export PATH="$(brew --prefix gcc)/bin:$PATH"
   export CC=gcc-15 CXX=g++-15
   export LIBRARY_PATH="$(brew --prefix gcc)/lib/gcc/15:$(brew --prefix gcc)/lib:${LIBRARY_PATH}"
   export DYLD_FALLBACK_LIBRARY_PATH="$(brew --prefix gcc)/lib/gcc/15:$(brew --prefix gcc)/lib:${DYLD_FALLBACK_LIBRARY_PATH}"
   ``` 
1. 將更新後的環境變數值傳遞到 Launcher，讓從 GUI 介面啟動 Emacs 也能讀到正確的環境變數值
   ```bash
   source ~/.zshrc
   launchctl setenv PATH "$PATH"
   launchctl setenv LIBRARY_PATH "$LIBRARY_PATH"
   launchctl setenv DYLD_FALLBACK_LIBRARY_PATH "$DYLD_FALLBACK_LIBRARY_PATH"
   ```
1. 清除舊有的 eln-cache 資料 ```~/.emacs.d/eln-cache/*```

下次啟動 Emacs，應該就不會有 ```libgccjit.dylib: No such file or directory``` 的錯誤訊息了。 