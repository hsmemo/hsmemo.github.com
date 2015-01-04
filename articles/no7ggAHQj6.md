---
layout: default
title: Class のロード/リンク/初期化 ： ロード処理 (1) ： ロード処理の開始点
---
[Up](nohUsh5oi7.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： ロード処理 (1) ： ロード処理の開始点

--- 
## 概要(Summary)
クラスのロード処理は以下の契機で開始される. (<= #TODO 他にないか??)

  * HotSpot の起動時
    
    HotSpot の起動時に, 幾つかの標準ライブラリのクラスがロードされる (java.lang.Object, java.lang.Thread, etc)
    (See: [here](notXYWwprj.html) for details).
    
  * メインクラスのロード時
    
    HotSpot の初期化後, java コマンド (launcher) によりメインクラスのロード処理が行われる
    (See: [here](noSl0AuhYv.html) for details).
    
  * 実行時コンスタントプールのシンボルの解決(resolution)時
    
    実行時コンスタントプール中のクラス名／インターフェース名の解決が行われた際, 
    対象がまだロードされていなければロード処理が実行される
    (See: [here](no7882_Jf.html) for details).
    
    (発生する件数としてはこのケースが一番多い
    (参考: [HotSpot Runtime Overview : VM Class Loading](http://openjdk.java.net/groups/hotspot/docs/RuntimeOverview.html#VM%20Class%20Loading|outline)))
    
  * クラスファイル検証中に, 検証のため他のクラスが必要になったとき.
    
    (See: [here](noYhX5khB4.html) for details)
    
  * クラスロードを伴う API が明示的に呼ばれたとき (java.lang.Class.forName(), java.lang.ClassLoader.loadClass(), Reflection APIs, etc)
    
    指定されたクラスが明示的にロードされる (See: [here](noYhX5khB4.html) for details).

## 備考(Notes)
* なお, HotSpot 内部の 「ClassLoader クラス」がいわゆる「ブートストラップ・クラスローダ」に相当すると思われる
  (注: java.lang.ClassLoader ではなく, C++ で書かれている ClasLoader クラス) (See: ClassLoader).
  最終的にはこの ClassLoader クラスの処理によってクラスのロード処理が行われる.
  
  ただし, ブートストラップ・クラスローダが直接ロードするのは標準ライブラリのクラス(rt.jar内のクラス等)だけで,
  それ以外のクラスは一旦 Java レベルのクラスローダーを経由した後で ClassLoader クラスでロードされる.

* なお, HotSpot のデフォルトのシステムクラスローダーは sun.misc.Launcher$AppClassLoader.
  このため, システムクラスローダーの ClassLoader.loadClass() 処理は
  sun.misc.Launcher$AppClassLoader.loadClass() に実装されている.




## Subcategories
* [(#TBD) Class のロード/リンク/初期化 ： ロード処理の開始点 ： HotSpot 起動時における標準ライブラリクラスのロード処理](notXYWwprj.html)
* [Class のロード/リンク/初期化 ： ロード処理の開始点 ： メインクラスのロード処理](noSl0AuhYv.html)
* [Class のロード/リンク/初期化 ： ロード処理の開始点 ： 実行時コンスタントプールのシンボルの解決(resolution)に伴うロード処理  ](no7882_Jf.html)
* [Class のロード/リンク/初期化 ： ロード処理の開始点 ： その他のロード処理](noYhX5khB4.html)



