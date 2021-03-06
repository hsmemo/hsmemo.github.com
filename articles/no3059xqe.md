---
layout: default
title: Class のロード/リンク/初期化 ： リンク処理 (2) ： リンク処理の流れ  
---
[Up](noX5hsnWQw.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： リンク処理 (2) ： リンク処理の流れ  

--- 
## 概要(Summary)
クラスのリンク処理は instanceKlass::link_class() (より正確には instanceKlass::link_class_impl()) で行われる.

リンク処理では, 以下の処理が行われる.

  * バイトコードの verification 処理 (See: [here](no7882amm.html) for details)

  * バイトコードの rewrite 処理, 及び rewrite に伴う再配置 (See: [here](no3059AfB.html) for details)

  * クラス内の vtable, itable の作成

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
instanceKlass::link_class()
-&gt; instanceKlass::link_class_impl()
   -&gt; (1) バイトコードの verification を行う
          -&gt; instanceKlass::verify_code()
              -&gt; Verifier::verify()
                 -&gt; (See: <a href="no7882amm.html">here</a> for details)

      (1) バイトコードの rewrite 処理を行う
          -&gt; instanceKlass::rewrite_class()
             -&gt; Rewriter::rewrite()
                -&gt; (See: <a href="no3059AfB.html">here</a> for details)

      (1) 
          -&gt; instanceKlass::relocate_and_link_methods()
             -&gt; Rewriter::relocate_and_link(instanceKlassHandle this_oop, TRAPS)
                -&gt; (See: <a href="no3059AfB.html">here</a> for details)

      (1) vtable 及び itable を生成する
          -&gt; klassVtable::initialize_vtable()
          -&gt; klassItable::initialize_itable()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### instanceKlass::link_class()
See: [here](no7517oAW.html) for details
### instanceKlass::link_class_impl()
See: [here](no18536Kvy.html) for details
### instanceKlass::verify_code()
See: [here](no18536QmJ.html) for details
### instanceKlass::rewrite_class()
See: [here](no18536r7c.html) for details
### instanceKlass::relocate_and_link_methods()
See: [here](no18536FlF.html) for details

### klassVtable::initialize_vtable()
(#Under Construction)
See: [here](no18536t22.html) for details
### klassVtable::initialize_from_super()
See: [here](no18536Wzn.html) for details
### arrayKlass::vtable()
See: [here](no18536uTO.html) for details
### instanceKlass::vtable()
See: [here](no18536iDb.html) for details
### klassVtable::copy_vtable_to()
See: [here](no18536W6b.html) for details
### klassVtable::update_inherited_vtable()
(#Under Construction)

### klassVtable::fill_in_mirandas()
(#Under Construction)


### klassItable::initialize_itable()
See: [here](no18536QFv.html) for details
### klassItable::initialize_itable_for_interface()
(#Under Construction)
See: [here](no1853645k.html) for details






