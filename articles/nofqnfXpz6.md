---
layout: default
title: Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： Template Interpreter での処理
---
[Up](no2114EV0.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： Template Interpreter での処理

--- 
## 概要(Summary)
Template Interpreter の場合, 
do_oop_store() 関数によって Write Barrier 処理を行うコードが生成される.

なお, do_oop_store() は現在は以下のパスで(のみ)呼び出されている.

* putfield バイトコード, および putstatic バイトコードの処理 (低速版)

```
  TemplateTable::putfield_or_static()
  -> do_oop_store()
```

* putfield バイトコード, および putstatic バイトコードの処理 (高速版)

```
  TemplateTable::fast_storefield()
  -> do_oop_store()
```

* aastore バイトコードの処理

```
  TemplateTable::aastore()
  -> do_oop_store()
```

## 備考(Notes)
なお G1GC の場合は, 
java.lang.ref.Reference.get() の処理 
(= InterpreterGenerator::generate_Reference_get_entry() が生成するコード)においても, 
MacroAssembler::g1_write_barrier_pre() が生成したコードによる処理が行われる
(See: [here](noyRxS6ail.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
```
-> do_oop_store() が生成するコード
   -> 使用する GC アルゴリズムが G1GC かどうかに応じて 2通りの処理が存在
      * G1GC 以外の場合:
        -> (1) 対象のポインタ値の書き換えを行う
               -> MacroAssembler::store_heap_oop() が生成するコード or
                  MacroAssembler::store_heap_oop_null()

           (2) 書き換え箇所に対応する Barrier Set 中の値を dirty にする
               -> MacroAssembler::store_check() が生成するコード
                  -> MacroAssembler::card_table_write() が生成するコード

      * G1GC の場合:
        -> (1) SATB 用の Write Barrier 処理を (その必要があれば) 実行する
               -> MacroAssembler::g1_write_barrier_pre() が生成するコード
                  -> * ConcurrentMarkThread が作業をしていない場合: (See: [here](no2935d4w.html) for details)
                       何もしない
                     * 現在の値(= 書き換え前の値)が NULL の場合:
                       何もしない
                     * それ以外の場合:
                       -> generate_satb_log_enqueue_if_necessary() が生成するコード
                          -> generate_satb_log_enqueue() が生成するコード
                             -> SATBMarkQueueSet::handle_zero_index_for_thread() (<= もうバッファに空きがない場合には呼び出す)
                                -> PtrQueue::handle_zero_index()
                                   -> ObjPtrQueue::should_enqueue_buffer()
                                   -> * ロックと関連づけられているバッファの場合:
                                        -> PtrQueue::locking_enqueue_completed_buffer()
                                           -> PtrQueueSet::enqueue_complete_buffer()
                                              -> Monitor::notify()
                                      * ロックと関連づけられていないバッファの場合:
                                        -> PtrQueueSet::process_or_enqueue_complete_buffer()
                                           -> * 自分で処理してしまう場合:
                                                -> DirtyCardQueueSet::mut_process_buffer()
                                              * それ以外の場合:
                                                -> PtrQueueSet::enqueue_complete_buffer()
                                                   -> (同上)
                                   -> PtrQueueSet::allocate_buffer()

           (2) 対象のポインタ値の書き換えを行う
               -> MacroAssembler::store_heap_oop() が生成するコード

           (3) 書き換え箇所に対応する Barrier Set 中の値を dirty にする
               -> MacroAssembler::g1_write_barrier_post() が生成するコード
                  -> * 同一 HeapRegion 内を指している場合:
                       何もしない
                     * それ以外の場合:
                       -> generate_dirty_card_log_enqueue_if_necessary() が生成するコード
                          -> generate_dirty_card_log_enqueue() が生成するコード
                             -> DirtyCardQueueSet::handle_zero_index_for_thread() (<= もうバッファに空きがない場合には呼び出す)
                                -> PtrQueue::handle_zero_index()
                                   -> (略) (同上)
                                      (ただし, ObjPtrQueue::should_enqueue_buffer() ではなく PtrQueue::should_enqueue_buffer() が呼び出される)
                                      -> PtrQueueSet::enqueue_complete_buffer()
                                         -> Monitor::notify()  (<= DirtyCardQ_CBL_mon に対して. これは ConcurrentG1RefineThread を起床させる処理 (See: [here](no2935dGZ.html) for details))

```

### x86_64 の場合
```
-> do_oop_store() が生成するコード
   -> 使用する GC アルゴリズムが G1GC かどうかに応じて 2通りの処理が存在
      * G1GC 以外の場合:
        -> (1) 対象のポインタ値の書き換えを行う
               -> MacroAssembler::store_heap_oop() が生成するコード

           (2) 書き換え箇所に対応する Barrier Set 中の値を dirty にする
               -> MacroAssembler::store_check()
                  -> MacroAssembler::store_check_part_1() が生成するコード
                  -> MacroAssembler::store_check_part_2() が生成するコード

      * G1GC の場合:
        -> (1) SATB 用の Write Barrier 処理を (その必要があれば) 実行する
               -> MacroAssembler::g1_write_barrier_pre()
                  -> * ConcurrentMarkThread が作業をしていない場合: (See: [here](no2935d4w.html) for details)
                       何もしない
                     * 現在の値(= 書き換え前の値)が NULL の場合:
                       何もしない
                     * それ以外の場合:
                       -> SharedRuntime::g1_wb_pre() (← もうバッファに空きがない場合には呼び出す)
                          -> PtrQueue::enqueue()     (← thread->satb_mark_queue() に対して呼び出す)
                             -> PtrQueue::enqueue_known_active()
                                -> PtrQueue::handle_zero_index()
                                   -> (同上)

           (2) 対象のポインタ値の書き換えを行う
               -> MacroAssembler::store_heap_oop() が生成するコード, または 
                  MacroAssembler::store_heap_oop_null() が生成するコード

           (2) 書き換え箇所に対応する Barrier Set 中の値を dirty にする
               -> MacroAssembler::g1_write_barrier_post() が生成するコード
                  -> SharedRuntime::g1_wb_post() (← もうバッファに空きがない場合には呼び出す)
                     -> PtrQueue::enqueue()      (← thread->dirty_card_queue() に対して呼び出す)
                        -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### do_oop_store() (sparc の場合)
See: [here](no3718BOz.html) for details

### MacroAssembler::card_write_barrier_post() (sparc の場合)
See: [here](no3718AiI.html) for details
### MacroAssembler::card_table_write() (sparc の場合)
See: [here](no3718NsO.html) for details

### MacroAssembler::g1_write_barrier_pre() (sparc の場合)
See: [here](no2935CxF.html) for details
### generate_satb_log_enqueue_if_necessary() (sparc の場合)
See: [here](no2935cFS.html) for details
### generate_satb_log_enqueue() (sparc の場合)
See: [here](no2935pPY.html) for details
### SATBMarkQueueSet::handle_zero_index_for_thread()
See: [here](no2935PCA.html) for details
### PtrQueue::handle_zero_index()
See: [here](no2935Q1e.html) for details
### ObjPtrQueue::should_enqueue_buffer()
See: [here](no2935qJr.html) for details
### PtrQueue::locking_enqueue_completed_buffer()
See: [here](no29352nG.html) for details
### PtrQueueSet::enqueue_complete_buffer()
See: [here](no2935DyM.html) for details
### PtrQueueSet::process_or_enqueue_complete_buffer()
See: [here](no29353Tx.html) for details
### DirtyCardQueueSet::mut_process_buffer()
See: [here](no2935pdA.html) for details
### PtrQueueSet::allocate_buffer()
See: [here](no2935Q8S.html) for details

### MacroAssembler::g1_write_barrier_post() (sparc の場合)
See: [here](no2935P7L.html) for details
### generate_dirty_card_log_enqueue_if_necessary() (sparc の場合)
See: [here](no2935cMG.html) for details
### generate_dirty_card_log_enqueue() (sparc の場合)
See: [here](no2935pWM.html) for details
### DirtyCardQueueSet::handle_zero_index_for_thread()
See: [here](no29352gS.html) for details
### PtrQueue::should_enqueue_buffer()
See: [here](no2935d_k.html) for details

### do_oop_store() (x86_64 の場合)
See: [here](no3718zXC.html) for details
### MacroAssembler::store_check() (x86_64 の場合)
See: [here](no37180Kh.html) for details
### MacroAssembler::store_check_part_1() (x86_64 の場合)
See: [here](no3718BVn.html) for details
### MacroAssembler::store_check_part_2() (x86_64 の場合)
See: [here](no3718Oft.html) for details
### MacroAssembler::g1_write_barrier_pre() (x86_64 の場合)
See: [here](no31977HDh.html) for details
### SharedRuntime::g1_wb_pre()
See: [here](no31977UNn.html) for details
### PtrQueue::enqueue()
See: [here](no31977uhz.html) for details
### PtrQueue::enqueue_known_active()
See: [here](no31977grC.html) for details
### MacroAssembler::g1_write_barrier_post() (x86_64 の場合)
See: [here](no31977t1I.html) for details
### SharedRuntime::g1_wb_post()
See: [here](no31977hXt.html) for details






