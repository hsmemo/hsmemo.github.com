---
layout: default
title: Serviceability 機能 ： OS のトレーサ機能との連携 ： (おまけ) SystemTap との連携
---
[Up](noYz1ICuBU.html) [Top](../index.html)

#### Serviceability 機能 ： OS のトレーサ機能との連携 ： (おまけ) SystemTap との連携

--- 
## 概要(Summary)
(この機能は OpenJDK 7u0 の正式な機能ではない)

Red Hat 社が IcedTea 向けに DTrace 連携機能を移植したもの. 
DTrace の代わりに Linux の ユーザーランド用 SystemTap を用いている.

### ソースコードの確認方法
ソースコード(というかパッチ)は OpenJDK 7u0 にはまだ入っていないため, IcedTea のソースで確認.
IcedTea のソースコードは以下のようにして入手 (確認日時 2012/03/19. 確認時点での changeset は "e909b2c85913").

````
  hg clone http://icedtea.classpath.org/hg/icedtea
````

### 関連するパッチ
以下のパッチが該当すると思われる.
主な変更箇所は systemtap.patch に入っている. 他の二つは DTrace の使用箇所に関して gcc 対応やバグの微修正をしただけである模様 (分量も他の二つは小さい).

  * icedtea/patches/systemtap.patch
  * icedtea/patches/systemtap-gcc-4.5.patch
  * icedtea/patches/systemtap-alloc-size-workaround.patch

### パッチの内容
systemtap.patch 自体が 160 行程度しかなく大きな変更はない
(実際問題として, DTRACE_PROBE() マクロにまで落ちれば Linux 版 sys/sdt.h のインラインアセンブラに任せるだけなので, 変更すべき箇所自体がほとんどないと思われる).

パッチの概要は以下の通り.

  * ビルド時に DTrace 機能を有効にする条件を,
    "#if defined(SOLARIS) && defined(DTRACE_ENABLED)" から "#ifdef DTRACE_ENABLED" に変更

  * HS_DTRACE_PROBE*() マクロが, HS_DTRACE_PROBE_FN() マクロではなく, 
    DTRACE_PROBE*() マクロに展開されるように変更.
    
    (元の hotspot/src/share/vm/utilities/dtrace.hpp では
    それぞれの HS_DTRACE_PROBE*() マクロは最終的に HS_DTRACE_PROBE_FN() マクロになり,
    そこから `__dtrace_*__*()` という関数コールに展開される.)

  * DTrace では (`__dtrace_*__*()` という関数コールに落ちる関係上, 使用箇所では `__dtrace_*__*()` 関数の型を宣言をしておく必要があり)
    HS_DTRACE_PROBE_DECL_N() や HS_DTRACE_PROBE_CDECL_N() というマクロがその型宣言に展開されていたが,
    Linux 版では (型宣言が不要なので) これらのマクロは空文字列に展開する.

  * 元々の DTRACE_PROBE* マクロは引数は5個までだったが, 引数6~10個までのバージョンが追加されている.
    (DTRACE_PROBE6(), DTRACE_PROBE7(), ..., DTRACE_PROBE10())
    
    (<= 元々 5個なのは「sparc でレジスタ渡しできるのが 5個までだから」という理由だったと思うが, Linux では sparc の重要性は低そうなのでまぁそれもありか)

  * (ついでに JNI の Set*Field と SetStatic*Field の entry 部分も変更しているが, これは内容的には同じに見える)

    (FP_SELECT_* というマクロによって引数が1つ増えるか増えないか, という部分の書き方が変わっている.)
     単に gcc がエラーを吐いたから, とかだろうか??)

  * (コピーライトに "* Copyright 2009 Red Hat, Inc." が追加されている)







