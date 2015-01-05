---
layout: default
title: Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： その他の箇所での処理 (JNI, etc)
---
[Up](no2114EV0.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： その他の箇所での処理 (JNI, etc)

--- 
#Under Construction

## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
-&gt; BarrierSet::write_ref_field_pre()
   -&gt; Barrier Set 種別や GC アルゴリズム, 変更対象等に応じて以下のどれかを行う.

      * Barrier Set の種別が CardTableModRef の場合: 
        * GC アルゴリズムが G1GC の場合:

          -&gt; G1SATBCardTableModRefBS::inline_write_ref_field_pre()
             -&gt; G1SATBCardTableModRefBS::write_ref_field_pre_static()
                -&gt; G1SATBCardTableModRefBS::enqueue()
                   -&gt; PtrQueue::enqueue()
                      -&gt; (同上)

        * GC アルゴリズムがそれ以外の場合:

          -&gt; CardTableModRefBS::inline_write_ref_field_pre()
      
      * CardTableModRef ではない場合:
        * GC アルゴリズムが G1GC の場合:
          * 変更対象が oop* の場合:

            -&gt; G1SATBCardTableModRefBS::write_ref_field_pre_work(oop* field, oop new_val)
               -&gt; G1SATBCardTableModRefBS::inline_write_ref_field_pre()()
                  -&gt; (同上)

          * 変更対象が narrowOop* の場合:

            -&gt; G1SATBCardTableModRefBS::write_ref_field_pre_work(narrowOop* field, oop new_val)
               -&gt; G1SATBCardTableModRefBS::inline_write_ref_field_pre()()
                  -&gt; (同上)

        * GC アルゴリズムがそれ以外の場合:
          * 変更対象が oop* の場合:

            -&gt; BarrierSet::write_ref_field_pre_work(oop* field, oop new_val)

          * 変更対象が narrowOop* の場合:

            -&gt; BarrierSet::write_ref_field_pre_work(narrowOop* field, oop new_val)

-&gt; BarrierSet::write_ref_field()
   -&gt; Barrier Set 種別や GC アルゴリズム, 変更対象等に応じて以下のどれかを行う.

      * Barrier Set の種別が CardTableModRef の場合: 
        
        -&gt; CardTableModRefBS::inline_write_ref_field()
      
      * CardTableModRef ではない場合:
        * GC アルゴリズムが G1GC の場合:
          
          -&gt; G1SATBCardTableLoggingModRefBS::write_ref_field_work()
             -&gt; PtrQueue::enqueue()

        * GC アルゴリズムがそれ以外の場合:
          
          -&gt; CardTableModRefBS::write_ref_field_work()
             -&gt; CardTableModRefBS::inline_write_ref_field()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
(#Under Construction)








