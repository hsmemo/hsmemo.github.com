---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interp_masm_sparc.cpp
### 説明(description)
// Test ImethodDataPtr.  If it is null, continue at the specified label



### 名前(function name)
```
void InterpreterMacroAssembler::test_method_data_pointer(Label& zero_continue) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(ProfileInterpreter, "must be profiling interpreter");

  {- -------------------------------------------
  (1) コード生成:
      「ImethodDataPtr レジスタの値が 0 (NULL) であれば, 
       引数で指定された zero_continue ラベルにジャンプする」
      ---------------------------------------- -}

	#ifdef _LP64
	  bpr(Assembler::rc_z, false, Assembler::pn, ImethodDataPtr, zero_continue);
	#else
	  tst(ImethodDataPtr);
	  br(Assembler::zero, false, Assembler::pn, zero_continue);
	#endif
	  delayed()->nop();
	}
	
```


