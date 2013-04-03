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
```
-> BarrierSet::write_ref_field_pre()
   -> Barrier Set 種別や GC アルゴリズム, 変更対象等に応じて以下のどれかを行う.

      * Barrier Set の種別が CardTableModRef の場合: 
        * GC アルゴリズムが G1GC の場合:

          -> G1SATBCardTableModRefBS::inline_write_ref_field_pre()
             -> G1SATBCardTableModRefBS::write_ref_field_pre_static()
                -> G1SATBCardTableModRefBS::enqueue()
                   -> PtrQueue::enqueue()
                      -> (同上)

        * GC アルゴリズムがそれ以外の場合:

          -> CardTableModRefBS::inline_write_ref_field_pre()
      
      * CardTableModRef ではない場合:
        * GC アルゴリズムが G1GC の場合:
          * 変更対象が oop* の場合:

            -> G1SATBCardTableModRefBS::write_ref_field_pre_work(oop* field, oop new_val)
               -> G1SATBCardTableModRefBS::inline_write_ref_field_pre()()
                  -> (同上)

          * 変更対象が narrowOop* の場合:

            -> G1SATBCardTableModRefBS::write_ref_field_pre_work(narrowOop* field, oop new_val)
               -> G1SATBCardTableModRefBS::inline_write_ref_field_pre()()
                  -> (同上)

        * GC アルゴリズムがそれ以外の場合:
          * 変更対象が oop* の場合:

            -> BarrierSet::write_ref_field_pre_work(oop* field, oop new_val)

          * 変更対象が narrowOop* の場合:

            -> BarrierSet::write_ref_field_pre_work(narrowOop* field, oop new_val)

-> BarrierSet::write_ref_field()
   -> Barrier Set 種別や GC アルゴリズム, 変更対象等に応じて以下のどれかを行う.

      * Barrier Set の種別が CardTableModRef の場合: 
        
        -> CardTableModRefBS::inline_write_ref_field()
      
      * CardTableModRef ではない場合:
        * GC アルゴリズムが G1GC の場合:
          
          -> G1SATBCardTableLoggingModRefBS::write_ref_field_work()
             -> PtrQueue::enqueue()

        * GC アルゴリズムがそれ以外の場合:
          
          -> CardTableModRefBS::write_ref_field_work()
             -> CardTableModRefBS::inline_write_ref_field()
```

## 処理の流れ (詳細)(Execution Flows : Details)
(#Under Construction)








