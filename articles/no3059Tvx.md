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
void TemplateTable::invokeinterface_object_method(Register RklassOop,
                                                  Register Rcall,
                                                  Register Rret,
                                                  Register Rflags) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Register Rscratch = G4_scratch;
	  Register Rindex = Lscratch;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(Rscratch, Rindex, Rret);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label notFinal;
	
  {- -------------------------------------------
  (1) コード生成:
      「呼び出し対象のメソッドに final 修飾子が付いているかどうかを確認.
        付いていなければ notFinal ラベルに分岐.」
      ---------------------------------------- -}

	  // Check for vfinal
	  __ set((1 << ConstantPoolCacheEntry::vfinalMethod), Rscratch);
	  __ btst(Rflags, Rscratch);
	  __ br(Assembler::zero, false, Assembler::pt, notFinal);
	  __ delayed()->nop();

  {- -------------------------------------------
  (1) (以下は, final 修飾子が付いていた場合のコード生成)
      ---------------------------------------- -}
	
    {- -------------------------------------------
  (1.1) コード生成:
        「method data pointer (mdp) の値を更新しておく」
        ---------------------------------------- -}

	  __ profile_final_call(O4);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「実際の呼び出し処理を行う」
        ---------------------------------------- -}

	  // do the call - the index (f2) contains the methodOop
	  assert_different_registers(G5_method, Gargs, Rcall);
	  __ mov(Rindex, G5_method);
	  __ call_from_interpreter(Rcall, Gargs, Rret);

  {- -------------------------------------------
  (1) (ここまでが, final 修飾子が付いていた場合)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下は, final 修飾子が付いていなかった場合のコード生成)
      ---------------------------------------- -}

	  __ bind(notFinal);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「method data pointer (mdp) の値を更新しておく」
        ---------------------------------------- -}

	  __ profile_virtual_call(RklassOop, O4);

    {- -------------------------------------------
  (1.1) コード生成:
        「実際の呼び出し処理を行う」
        ---------------------------------------- -}

	  generate_vtable_call(RklassOop, Rindex, Rret);

  {- -------------------------------------------
  (1) (ここまでが, final 修飾子が付いていなかった場合)
      ---------------------------------------- -}

	}
	
```


