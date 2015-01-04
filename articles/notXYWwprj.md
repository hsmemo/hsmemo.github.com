---
layout: default
title: (#TBD) Class のロード/リンク/初期化 ： ロード処理の開始点 ： HotSpot 起動時における標準ライブラリクラスのロード処理
---
[Up](no7ggAHQj6.html) [Top](../index.html)

#### (#TBD) Class のロード/リンク/初期化 ： ロード処理の開始点 ： HotSpot 起動時における標準ライブラリクラスのロード処理

--- 
## 概要(Summary)
標準ライブラリのクラスの幾つかは HotSpot の起動処理中にロードされる (java.lang.Object, java.lang.Thread, 等).

(この処理は Threads::create_vm() 内で呼び出される各種の関数内で行われる模様. (<= 他にないか？#TODO))

## 処理の流れ (概要)(Execution Flows : Summary)
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> init_globals()
      -> universe2_init()
         -> Universe::genesis()
            -> SystemDictionary::initialize()
               -> SystemDictionary::initialize_preloaded_classes()
                  -> 
      -> universe_post_init()
         -> 

   -> initialize_class()  (※)
      -> (1) 対象クラスを取得する. まだロードされていなければロードも行う.
             -> SystemDictionary::resolve_or_fail()
                -> (See: [here](noIvSV0NZj.html) for details)

         (2) 対象クラスのリンクおよび初期化を行う
             -> instanceKlass::initialize()
                -> (See: [here](no9AAGw84F.html) for details)

(※) なお, この関数はクラスローダーとして「ブートストラップ・クラスローダ」を使用.
```

## 処理の流れ (詳細)(Execution Flows : Details)
### Threads::create_vm()
See: [here](no4230j8a.html) for details
### universe_post_init()
See: [here](no3269WqK.html) for details
### initialize_class()
See: [here](no26814_eP.html) for details






