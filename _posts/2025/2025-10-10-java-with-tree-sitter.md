---
layout: post
title: 在 Java 程式中使用 tree-sitter
categories: java

---

[Tree-sitter](https://tree-sitter.github.io/tree-sitter/) 是近年十分受歡迎的 parser 實作，特色為通用、快速、耐用且使用純 c11 完成，沒有任何外部相依性。同時對大多數語言都提供了介接用的界面。

最近剛好想自己寫點程式處理 [org-mode](https://orgmode.org) 的檔案，所以就用 Java 透過 tree-sitter 做了些處理，順便記錄一下在 Java 語言中使用 tree-sitter 需要注意的事情。


# 環境說明
- 作業系統: macOS 26.0.1
- 套件管理工具： homebrew


首先必須安裝 tree-sitter，如果需要 tree-sitter 的命令列工具，則需要額外安裝 tree-sitter-cli
```bash
$ brew install tree-sitter tree-sitter-cli
```

在我的環境上，安裝的是 tree-sitter 0.25.10 版，可以找到 `/opt/homebrew/Cellar/tree-sitter/0.25.10/lib/libtree-sitter.dylib` 這個檔案，接著還需要特定語言的 parser，這裡直接使用 nvim org-mode 外掛（plugin）中的 parser 函式庫 `~/.local/share/nvim/lazy/orgmode/parser/org.so`。

有了 `libtree-sitter.dylib` 跟 `org.so` 兩個函式庫後，就可以開始用 Java 程式操作 org 檔案了。

# 程式範例

接下來直接上程式，說明請參考細節

```java
// 載入 tree-sitter library
System.load("/opt/homebrew/Cellar/tree-sitter/0.25.10/lib/libtree-sitter.dylib");

// 用 org.so 建立 Tree-sitter 的 Language
// 1. 載入 navtive parser
Arena arena = Arena.ofAuto();
SymbolLookup symbols = SymbolLookup.libraryLookup(Path.of("`/.local/share/nvim/lazy/orgmode/parser/org.so"), arena);

// 2. 用 native parser symbol 建立 Language 物件
var address = symbols.find("tree_sitter_org").get(); 
var function = Linker.nativeLinker().downcallHandle(address, FunctionDescriptor.of(ValueLayout.ADDRESS.withTargetLayout(MemoryLayout.sequenceLayout(Long.MAX_VALUE, ValueLayout.JAVA_BYTE))));
Language language = new Language((MemorySegment) function.invokeExact());

// 3. 用 Language 建立 Parser
String text_to_parse = "...";
try ( Parser parser = new Parser(language)) {
// 4. 用 parser 剖析 text_to_parse
    try (Tree tree = parser.parse(org_text, InputEncoding.UTF_8).orElseThrow()) {
// 5. 處理剖析後產生的 Tree        
        IO.println(tree);
    }
}

```

其中 `var address = symbols.find("tree_sitter_org").get();` 這行程式在正式產品中，應該要用 `orElseThrow()` 作例外處理，而不是像範例程式一樣直接假設不會出問題。

這就是在 Java 中用 Tree-sitter 的最小限度需要的程式碼，上述範例沒有涵蓋使用剖析出來的 Tree 物件的部份，這部份就參考 [Java Tree-sitter biding](https://tree-sitter.github.io/java-tree-sitter/io/github/treesitter/jtreesitter/package-summary.html) 的 JavaDoc 文件就行了。

