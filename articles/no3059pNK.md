---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateTable_sparc.cpp

### 名前(function name)
```
void TemplateTable::lrem() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(ltos, ltos);
	
  {- -------------------------------------------
  (1) コード生成:
      「dividend を O2 レジスタにロード」
      ---------------------------------------- -}

	  // check for zero
	  __ pop_l(O2);

  {- -------------------------------------------
  (1) コード生成:
      「divisor(Otos_i)が 0 かどうかを確認する.
        0 の場合は ArithmeticException (See: [here](no3059cDE.html) for details).
        そうでなければ, 以下のどちらかを行うことで剰余計算を実施.
        * 64bit の場合
          X mod Y = X - (X/Y)*Y  なので, その通りに計算.
        * 32bit の場合
          SharedRuntime::lrem() を呼び出す               」
      ---------------------------------------- -}

	#ifdef _LP64
	  __ tst(Otos_l);
	  __ throw_if_not_xcc( Assembler::notZero, Interpreter::_throw_ArithmeticException_entry, G3_scratch);
	  __ sdivx(O2, Otos_l, Otos_l2);
	  __ mulx (Otos_l2, Otos_l, Otos_l2);
	  __ sub  (O2, Otos_l2, Otos_l);
	#else
	  __ orcc(Otos_l1, Otos_l2, G0);
	  __ throw_if_not_icc(Assembler::notZero, Interpreter::_throw_ArithmeticException_entry, G3_scratch);
	  __ call_VM_leaf(Lscratch, CAST_FROM_FN_PTR(address, SharedRuntime::lrem));
	#endif
	}
	
```


