---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateTable_x86_64.cpp
### 説明(description)

```
// Miscelaneous helper routines
// Store an oop (or NULL) at the address described by obj.
// If val == noreg this means store a NULL

```

### 名前(function name)
```
static void do_oop_store(InterpreterMacroAssembler* _masm,
                         Address obj,
                         Register val,
                         BarrierSet::Name barrier,
                         bool precise) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(val == noreg || val == rax, "parameter is just for looks");

  {- -------------------------------------------
  (1) barrier set の種別 (= GC アルゴリズムの種別) に応じて, 以下の3種類のケースが実装されている.
      * G1GC の場合 (BarrierSet::G1SATBCT または BarrierSet::G1SATBCTLogging の場合)
      * G1GC 以外の場合 (BarrierSet::CardTableModRef または BarrierSet::CardTableExtension の場合)
      * ??の場合 (BarrierSet::ModRef または BarrierSet::Other の場合)
      (なお, 正確には上記以外の場合というパスも書かれているが, 
       Should_Not_Reach_Here() になるだけ.)
    
      (sparc 版では BarrierSet::ModRef や BarrierSet::Other の場合も Should_Not_Reach_Here() だったが, なぜ違う?? #TODO)
      ---------------------------------------- -}

	  switch (barrier) {

  {- -------------------------------------------
  (1) (以下は G1GC の場合のパス)
      ---------------------------------------- -}

	#ifndef SERIALGC
	    case BarrierSet::G1SATBCT:
	    case BarrierSet::G1SATBCTLogging:
	      {

    {- -------------------------------------------
  (1.1) コード生成:
        「rdx レジスタに対象のオブジェクト(obj 引数)のアドレスをセットする」
        ---------------------------------------- -}

	        // flatten object address if needed
	        if (obj.index() == noreg && obj.disp() == 0) {
	          if (obj.base() != rdx) {
	            __ movq(rdx, obj.base());
	          }
	        } else {
	          __ leaq(rdx, obj);
	        }

    {- -------------------------------------------
  (1.1) コード生成:
        「MacroAssembler::g1_write_barrier_pre() が生成するコードにより, 
          SATB 用の Write Barrier 処理を (その必要があれば) 実行する.」
        ---------------------------------------- -}

	        __ g1_write_barrier_pre(rdx /* obj */,
	                                rbx /* pre_val */,
	                                r15_thread /* thread */,
	                                r8  /* tmp */,
	                                val != noreg /* tosca_live */,
	                                false /* expand_call */);

    {- -------------------------------------------
  (1.1) コード生成:
        「MacroAssembler::store_heap_oop() または 
          MacroAssembler::store_heap_oop_null() が生成するコードにより, 
          対象のポインタ値の書き換えを行う.
          (書き込むポインタ値が NULL でない場合には MacroAssembler::store_heap_oop(), 
          NULL の場合には MacroAssembler::store_heap_oop_null() を使用)
  
          また, 書き込むポインタ値が NULL でない場合には, 
          MacroAssembler::g1_write_barrier_post() が生成するコードにより, 
          書き換え箇所に対応する Barrier Set 中の値を dirty にする」
  
        (NULL の場合には, コード生成時点で記録の必要が無いと保証できる)
        ---------------------------------------- -}

	        if (val == noreg) {
	          __ store_heap_oop_null(Address(rdx, 0));
	        } else {
	          __ store_heap_oop(Address(rdx, 0), val);
	          __ g1_write_barrier_post(rdx /* store_adr */,
	                                   val /* new_val */,
	                                   r15_thread /* thread */,
	                                   r8 /* tmp */,
	                                   rbx /* tmp2 */);
	        }
	
	      }
	      break;
	#endif // SERIALGC

  {- -------------------------------------------
  (1) (以下は G1GC 以外の場合のパス)
      ---------------------------------------- -}

	    case BarrierSet::CardTableModRef:
	    case BarrierSet::CardTableExtension:
	      {

    {- -------------------------------------------
  (1.1) コード生成:
        「MacroAssembler::store_heap_oop() または 
          MacroAssembler::store_heap_oop_null() が生成するコードにより, 
          対象のポインタ値の書き換えを行う.
          (書き込むポインタ値が NULL でない場合には MacroAssembler::store_heap_oop(), 
          NULL の場合には MacroAssembler::store_heap_oop_null() を使用)
  
          また, 書き込むポインタ値が NULL でない場合には, 
          MacroAssembler::store_check() が生成するコードにより, 
          書き換え箇所に対応する Barrier Set 中の値を dirty にする」
  
        (NULL の場合には, コード生成時点で記録の必要が無いと保証できる)
        (なお, precise 引数が true の場合には, 
         base で指定された値に offset/index 分も足し込んだ正確なアドレスを記録する.
         そうでない場合には base の値だけを記録する)
        ---------------------------------------- -}

	        if (val == noreg) {
	          __ store_heap_oop_null(obj);
	        } else {
	          __ store_heap_oop(obj, val);
	          // flatten object address if needed
	          if (!precise || (obj.index() == noreg && obj.disp() == 0)) {
	            __ store_check(obj.base());
	          } else {
	            __ leaq(rdx, obj);
	            __ store_check(rdx);
	          }
	        }
	      }
	      break;

  {- -------------------------------------------
  (1) (以下は ?? の場合のパス)
      #TODO
      ---------------------------------------- -}

	    case BarrierSet::ModRef:
	    case BarrierSet::Other:
	      if (val == noreg) {
	        __ store_heap_oop_null(obj);
	      } else {
	        __ store_heap_oop(obj, val);
	      }
	      break;

  {- -------------------------------------------
  (1) (以下のパスに来ることはあり得ない)
      (もし来てしまったら HotSpot 内のバグ)
      ---------------------------------------- -}

	    default      :
	      ShouldNotReachHere();
	
	  }
	}
	
```


