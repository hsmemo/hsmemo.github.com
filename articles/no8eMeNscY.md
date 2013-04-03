---
layout: default
title: Method に関する処理 ： calling convention (呼び出し規約) ： sparc の場合
---
[Up](noxTQWbUKc.html) [Top](../index.html)

#### Method に関する処理 ： calling convention (呼び出し規約) ： sparc の場合

--- 
## 概要(Summary)
System V の psABI とほぼ同様.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table2863bAu -->
| Convention | Description |
|---|---|
| caller/callee register | System V psABI の通り (このため save/restore 命令で対応可能). |
| 引数 | 呼び出し先がインタープリタ実行される場合は, スタック上に積んで渡す (See: ...). JIT コンパイルされている場合は, レジスタで渡す (See: ...). |
| リターンアドレス | System V psABI の通り, O7/I7 を使用. |
| 返値 | System V psABI の通り, O0/I0, F0 を使用 (ただし 32bit 環境で long 値を返すケースについては, O1/I1 も使用) |
| SP のずれへの対応 | callee 側のエントリ処理や i2c/c2i stub により SP が勝手にずらされてしまった場合については, 全て interpreter 側で対処している(※) |
<!-- END RECEIVE ORGTBL table2863bAu -->

<!-- 
#+ORGTBL: SEND table2863bAu orgtbl-to-gfm :no-escape t
| Convention             | Description                                                                                                                               |
|------------------------+-------------------------------------------------------------------------------------------------------------------------------------------|
| caller/callee register | System V psABI の通り (このため save/restore 命令で対応可能).                                                                             |
| 引数                   | 呼び出し先がインタープリタ実行される場合は, スタック上に積んで渡す (See: ...). JIT コンパイルされている場合は, レジスタで渡す (See: ...). |
| リターンアドレス       | System V psABI の通り, O7/I7 を使用.                                                                                                      |
| 返値                   | System V psABI の通り, O0/I0, F0 を使用 (ただし 32bit 環境で long 値を返すケースについては, O1/I1 も使用)                                 |
| SP のずれへの対応      | callee 側のエントリ処理や i2c/c2i stub により SP が勝手にずらされてしまった場合については, 全て interpreter 側で対処している(※)          |
-->

(※) (interpreter 側の呼び出し時には return 後にずれを補正し,
      逆に interpreter が呼び出された時には, return 時に普通の restore ではなくずれを考慮した restore を行う.
      これにより, compiled code 側ではシンプルに save/restore するだけで対処できている模様)
     (以下の O5_savedSP 及び Llast_SP 参照)

## 備考(Notes)
  * ただし, G2 については, JVM 内部では常時 Thread を指すように定めている.


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
    // R_G2: reserved by HotSpot to the TLS register (invariant within Java)
```

  * O7 については, C2 内では適当に scratch 的に使う (register allocation の対象にしないようにしている).


```
    ((cite: hotspot/src/cpu/sparc/vm/sparc.ad))
    // R_O7: Used as a temp in many encodings
```

  * callee がインタープリタの場合の Gargs については,
    caller 側が正しい位置に設定するように見えるが正しいか?? (#TODO)






