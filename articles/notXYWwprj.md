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
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; init_globals()
      -&gt; universe2_init()
         -&gt; Universe::genesis()
            -&gt; SystemDictionary::initialize()
               -&gt; SystemDictionary::initialize_preloaded_classes()
                  -&gt; 
      -&gt; universe_post_init()
         -&gt; 

   -&gt; initialize_class()  (※)
      -&gt; (1) 対象クラスを取得する. まだロードされていなければロードも行う.
             -&gt; SystemDictionary::resolve_or_fail()
                -&gt; (See: <a href="noIvSV0NZj.html">here</a> for details)

         (2) 対象クラスのリンクおよび初期化を行う
             -&gt; instanceKlass::initialize()
                -&gt; (See: <a href="no9AAGw84F.html">here</a> for details)

(※) なお, この関数はクラスローダーとして「ブートストラップ・クラスローダ」を使用.
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### Threads::create_vm()
See: [here](no4230j8a.html) for details
### universe_post_init()
See: [here](no3269WqK.html) for details
### initialize_class()
See: [here](no26814_eP.html) for details






