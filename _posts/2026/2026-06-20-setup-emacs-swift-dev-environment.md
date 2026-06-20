---
layout: post
title: "Setup emacs with xcode"
date: 2026-06-20 13:47:31 +0800
author: Zmb
categories: [Emacs, macOS]
tags: [emacs, swift, xcode, projectile]
---

最近因為需要個 macOS 的桌面小工具，打算利用這個機會學一下 Swift 跟 macOS 的軟體開發。雖然在這個 AI 時代，手刻程式的人已經不多了，但學新東西本來就是件有趣的事，當然要利用機會玩一下了。

搜尋了一下用 Emacs 開發 macOS 應用的方式，了解目前基本上是用 Emacs 作為編輯環境，專案的建置等工作還是得透過 Xcode 來做，而 `sourcekit-lsp` 是唯一能提供正確語法補全、類別檢查、跳轉定義的工具。

所以花了點時間設定自己的工作環境，最後在 Emacs 上的設定是 `eglot` + `swift-mode` + `projectile`。沒有選用 `swift-ts-mode` 是因為目前似乎還無法正確的處理 keyword 部份，顯示的內容還是少了一點。

## 安裝 sourcekit-lsp

- `xcode-select --install`

## 設定 Emacs

### 設定 emacs + eglot + swift-ts-mode

目前這個設定下，`swift-ts-mode` 處理 syntax highlight 上還有問題，可能無法正常的處理 Swift 的 keywords。

#### Install tree-sitter swift grammar for emacs

直接用 Emacs 的 `treesit-install-language-grammar` 會失敗，只好用手動安裝的方式：

```bash
$ git clone https://github.com/rechsteiner/swift-ts-mode
$ cd swift-ts-mode
$ tree-sitter generate
$ cp swift.dylib ~/.emacs.d/tree-sitter/libtree-sitter-swift.dylib
```

#### Setup Emacs

```elisp
(use-package eglot
  :ensure t
  ;; optimize eglot
  (setq eglot-sync-connect nil)  ;; async connection, prevent jam emacs
  (setq eglot-connect-timeout 60)   ;; increase timeout for large project
  )

(use-package swift-ts-mode
  :ensure t
  :mode ("\\.swift\\'" . swift-ts-mode) ;; 自動將 .swift 檔案關聯到此模式
  :config
  ;; 連結 Eglot 與 SourceKit-LSP
  (add-to-list 'eglot-server-programs
               '(swift-ts-mode . ("sourcekit-lsp")))
  (add-hook 'swift-ts-mode-hook 'eglot-ensure))
```

##### Status: Cannot handle font lock for keyword

```text
- `keyword' for swift.
⛔ Warning (treesit-font-lock-rules-mismatch): Emacs cannot compile every font-lock rules because a mismatch between the grammar and the rules.  This is most likely due to a mismatch between the font-lock rules defined by the major mode and the tree-sitter grammar.

This error can be fixed by either downgrading the grammar (tree-sitter-swift) on your system, or upgrading the major mode package.  The following are the temporarily disabled features:
```

### 設定 Emacs + eglot + swift-mode

這個設定相對單純，只需要在 `init.el` 設好 `eglot` + `swift-mode` 就行了，這也是目前我使用的設定。

```elisp
(use-package eglot
  :ensure t
  :defer
  :hook ((swift-mode . eglot-ensure))
  ;; optimize eglot
  :config
   ;; 告訴 Eglot 如何啟動 sourcekit-lsp
  (add-to-list 'eglot-server-programs
               '(swift-mode . ("sourcekit-lsp")))

  (setq eglot-sync-connect nil)  ;; async connection, prevent jam emacs
  (setq eglot-connect-timeout 60)   ;; increase timeout for large project
  )

(use-package swift-mode
  :ensure t
  :mode ("\\.swift\\'" . swift-mode))
```

### Setup Projectile

額外設定 projectile，改善 Emacs 下的專案工作環境：

- Emacs: `package-install projectile`
- Elisp setup (`init.el`):

```elisp
(use-package projectile
  :ensure t
  :init
  (setq projectile-project-search-path '("~/Projects/"))
  :config
  ;; I typically use this keymap prefix on macOS
  (define-key projectile-mode-map (kbd "s-p") 'projectile-command-map)
  ;; Recommended keymap prefix on Windows/Linux
  ;(define-key projectile-mode-map (kbd "C-c p") 'projectile-command-map)
  (global-set-key (kbd "C-c p") 'projectile-command-map)

  (projectile-mode +1))

;; If you use vertico
(use-package vertico
  :init
  (vertico-mode +1))
```
