---
layout: default
title: JNI の処理 ： native method の処理
---
[Up](noTiWFvTIw.html) [Top](../index.html)

#### JNI の処理 ： native method の処理

--- 
## 概要(Summary)
native method の処理は, 主に以下の3つからなる.

  * native method のダイナミックロード処理 (= java.lang.System.loadLibrary() の処理)
  * native method のダイナミックリンク処理 ("resolution 処理")
  * native method の呼び出し処理

## 備考(Notes)
ダイナミックロード済みのライブラリについては, 以下の2種類の管理方式がある.

* システムクラス用の libjava, および JVMTI agent としてロードしたライブラリの場合:

  HotSpot 内で管理する.

* それ以外のネイティブライブラリの場合:

  Java の ClassLoader オブジェクトが, それぞれ自分がロードしたネイティブライブラリを
  NativeLibrary オブジェクトという形で管理している.




## Subcategories
* [JNI の処理 ： native method の処理 ： native method の dynamic loading 処理(java.lang.System.loadLibrary() の処理) (JNI_OnLoad() の呼び出し処理も含む)](nobPjIPPvn.html)
* [JNI の処理 ： native method の処理 ： native method の dynamic linking 処理  ](no3059oyc.html)
* [JNI の処理 ： native method の処理 ： native method の呼び出し処理](noF_QFKdsW.html)



