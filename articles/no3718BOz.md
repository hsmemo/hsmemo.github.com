---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateTable_sparc.cpp
### 説明(description)

```
// Do an oop store like *(base + index + offset) = val
// index can be noreg,
```

### 名前(function name)
```
static void do_oop_store(InterpreterMacroAssembler* _masm,
                         Register base,
                         Register index,
                         int offset,
                         Register val,
                         Register tmp,
                         BarrierSet::Name barrier,
                         bool precise) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(tmp != val && tmp != base && tmp != index, "register collision");
	  assert(index == noreg || offset == 0, "only one offset");

  {- -------------------------------------------
  (1) (この関数の処理は, barrier set の種別 (= GC アルゴリズムの種別) に応じて2通りのパスが存在
       * G1GC の場合    (BarrierSet::G1SATBCT または BarrierSet::G1SATBCTLogging の場合)
       * G1GC 以外の場合 (BarrierSet::CardTableModRef または BarrierSet::CardTableExtension の場合))
  
      (なお, 正確には上記以外の場合 (BarrierSet::ModRef, BarrierSet::Other, またはそれ以外) というパスも書かれているが, 
       Should_Not_Reach_Here() になるだけ)
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
        「MacroAssembler::g1_write_barrier_pre() が生成するコードにより, 
          SATB 用の Write Barrier 処理を (その必要があれば) 実行する.」
        ---------------------------------------- -}

	        // Load and record the previous value.
	        __ g1_write_barrier_pre(base, index, offset,
	                                noreg /* pre_val */,
	                                tmp, true /*preserve_o_regs*/);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「MacroAssembler::store_heap_oop() が生成するコードにより, 
          対象のポインタ値の書き換えを行う」
        ---------------------------------------- -}

	        if (index == noreg ) {
	          assert(Assembler::is_simm13(offset), "fix this code");
	          __ store_heap_oop(val, base, offset);
	        } else {
	          __ store_heap_oop(val, base, index);
	        }
	
    {- -------------------------------------------
  (1.1) コード生成: (ただし, 書き込み元が G0 レジスタ(= 0) の場合は生成しない)
        「MacroAssembler::g1_write_barrier_pre() が生成するコードにより,
          書き換え箇所に対応する Barrier Set 中の値を dirty にする.」
    
        (G0 レジスタの場合に生成しないのは, コード生成時点で NULL (= 記録の必要が無い) と保証できるため)
        (なお, precise 引数が true の場合には, 
         base で指定された値に offset/index 分も足し込んだ正確なアドレスを記録する.
         そうでない場合には base の値だけを記録する)
        ---------------------------------------- -}

	        // No need for post barrier if storing NULL
	        if (val != G0) {
	          if (precise) {
	            if (index == noreg) {
	              __ add(base, offset, base);
	            } else {
	              __ add(base, index, base);
	            }
	          }
	          __ g1_write_barrier_post(base, val, tmp);
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
        「MacroAssembler::store_heap_oop() が生成するコードにより, 
          対象のポインタ値の書き換えを行う」
        ---------------------------------------- -}

	        if (index == noreg ) {
	          assert(Assembler::is_simm13(offset), "fix this code");
	          __ store_heap_oop(val, base, offset);
	        } else {
	          __ store_heap_oop(val, base, index);
	        }

    {- -------------------------------------------
  (1.1) コード生成: (ただし, 書き込み元が G0 レジスタ(= 0) の場合は生成しない)
        「MacroAssembler::card_write_barrier_post() が生成するコードにより, 
          書き換え箇所に対応する Barrier Set 中の値を dirty にする.」
  
        (G0 レジスタの場合に生成しないのは, コード生成時点で NULL (= 記録の必要が無い) と保証できるため)
        (なお, precise 引数が true の場合には, 
         base で指定された値に offset/index 分も足し込んだ正確なアドレスを記録する.
         そうでない場合には base の値だけを記録する)
        ---------------------------------------- -}

	        // No need for post barrier if storing NULL
	        if (val != G0) {
	          if (precise) {
	            if (index == noreg) {
	              __ add(base, offset, base);
	            } else {
	              __ add(base, index, base);
	            }
	          }
	          __ card_write_barrier_post(base, val, tmp);
	        }
	      }
	      break;

  {- -------------------------------------------
  (1) (以下のパスに来ることはあり得ない)
      (もし来てしまったら HotSpot 内のバグ)
      ---------------------------------------- -}

	    case BarrierSet::ModRef:
	    case BarrierSet::Other:
	      ShouldNotReachHere();
	      break;
	    default      :
	      ShouldNotReachHere();
	
	  }
	}
	
```


