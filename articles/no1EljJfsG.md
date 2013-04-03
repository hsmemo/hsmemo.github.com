---
layout: default
title: Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： C1 JIT Compiler が生成したコードでの処理
---
[Up](no2114EV0.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： Write Barrier 処理 ： C1 JIT Compiler が生成したコードでの処理

--- 
#Under Construction

## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
```
-> G1PreBarrierStub::emit_code() が生成するコード
   -> Runtime1::generate_blob_for() が生成するコード (← id 引数として g1_pre_barrier_slow_id を指定)
      -> Runtime1::generate_code_for() が生成するコード (← ただし id 引数が g1_pre_barrier_slow_id の場合)
         -> SATBMarkQueueSet::handle_zero_index_for_thread() (← もうバッファに空きがない場合には呼び出す)
            -> (同上)
  
-> G1UnsafeGetObjSATBBarrierStub::emit_code() が生成するコード
   -> Runtime1::generate_blob_for() が生成するコード (← id 引数として g1_pre_barrier_slow_id を指定)
      -> Runtime1::generate_code_for() が生成するコード (← ただし id 引数が g1_pre_barrier_slow_id の場合)
         -> SATBMarkQueueSet::handle_zero_index_for_thread() (← もうバッファに空きがない場合には呼び出す)
            -> (同上)

-> G1PostBarrierStub::emit_code() が生成するコード
   -> Runtime1::generate_blob_for() が生成するコード (← id 引数として g1_post_barrier_slow_id を指定)
      -> Runtime1::generate_code_for() が生成するコード (← ただし id 引数が g1_post_barrier_slow_id の場合)
         -> DirtyCardQueueSet::handle_zero_index_for_thread() (← もうバッファに空きがない場合には呼び出す)
            -> (同上)
```

### x86_64 の場合
```
-> G1PreBarrierStub::emit_code() が生成するコード
   -> Runtime1::generate_blob_for() が生成するコード (← id 引数として g1_pre_barrier_slow_id を指定)
      -> Runtime1::generate_code_for() が生成するコード (← ただし id 引数が g1_pre_barrier_slow_id の場合)
         -> SharedRuntime::g1_wb_pre() (← もうバッファに空きがない場合には呼び出す)
            -> (同上)
  
-> G1UnsafeGetObjSATBBarrierStub::emit_code() が生成するコード
   -> Runtime1::generate_blob_for() が生成するコード (← id 引数として g1_pre_barrier_slow_id を指定)
      -> Runtime1::generate_code_for() が生成するコード (← ただし id 引数が g1_pre_barrier_slow_id の場合)
         -> SharedRuntime::g1_wb_pre() (← もうバッファに空きがない場合には呼び出す)
            -> (同上)

-> G1PostBarrierStub::emit_code() が生成するコード
   -> Runtime1::generate_blob_for() が生成するコード (← id 引数として g1_post_barrier_slow_id を指定)
      -> Runtime1::generate_code_for() が生成するコード (← ただし id 引数が g1_post_barrier_slow_id の場合)
         -> SharedRuntime::g1_wb_post() (← もうバッファに空きがない場合には呼び出す)
            -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)
(#Under Construction)







