---
layout: default
title: OpenJDK のビルドについて(Build Instructions)
---
[Up](../index.html) [Top](../index.html)

#### OpenJDK のビルドについて(Build Instructions)

--- 
## 概要(Summary)
(#Under Construction)

## 備考(Notes)
ビルド時の設定を変えることで, デバッグ情報を付けることもできる (内部をいじる時には jvmg 等が便利).

 * product   : 通常時のビルド設定. -O 付き(最適化付き)でビルド, assert は全てオフ, -DPRODUCT 付き (PRODUCT が #define された状態でビルド)
 * jvmg	     : libjvm を -g 付きでビルド (libjvm_g が生成される)
 * fastdebug : -O 付き(最適化付き)でビルド, assert は全てオン.
 * optimized : -O 付き(最適化付き)でビルド, assert は全てオフ.  (#未確認)
 * debug     : gamma launcher を -g 付きでビルド (gamma_g)  (#未確認)
 * profiled  : gprof 付きでビルド  (#未確認)


```
    ((cite: hotspot/make/linux/Makefile))
    # debug*     - "thin" libjvm_g - debug info linked into the gamma_g launcher
    # fastdebug* - optimized compile, but with asserts enabled
    # jvmg*      - "fat" libjvm_g - debug info linked into libjvm_g.so
    # optimized* - optimized compile, no asserts
    # profiled*  - gprof
    # product*   - the shippable thing:  optimized compile, no asserts, -DPRODUCT
```




## Subcategories
* [ソースコードのビルド方法(How to Build)](noEDlz4EKK.html)
* [ビルド時に有効な環境変数/オプション(Useful Environment Variables and Options)](noIrJHHzUM.html)
* [(#TBD) Makefile の内部構造](nojB2Sq0ue.html)
* [(参考) ビルド後に生成されるファイル  ](no7882LhG.html)
* [(おまけ) hotspot/src/share/tools/ 以下のビルド方法](nokM03O0BP.html)



