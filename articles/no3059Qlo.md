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
void TemplateTable::idiv() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // %%%%% Later: ForSPARC/V7 call .sdiv library routine,
	  // %%%%% Use ldsw...sdivx on pure V9 ABI. 64 bit safe.
	
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(itos, itos);

  {- -------------------------------------------
  (1) コード生成:
      「dividend を O1 レジスタにロード」
      ---------------------------------------- -}

	  __ pop_i(O1); // get 1st op
	
  {- -------------------------------------------
  (1) コード生成:
      「Y レジスタに値をセットする.
        (dividend が負数であれば, 全てのビットを 1 にした値をセット.
        そうでなければ 0 をセットする)」
      ---------------------------------------- -}

	  // Y contains upper 32 bits of result, set it to 0 or all ones
	  __ wry(G0);
	  __ mov(~0, G3_scratch);
	
	  __ tst(O1);
	     Label neg;
	  __ br(Assembler::negative, true, Assembler::pn, neg);
	  __ delayed()->wry(G3_scratch);
	  __ bind(neg);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (この ok はどこからも使われていないが...??)
      ---------------------------------------- -}

	     Label ok;

  {- -------------------------------------------
  (1) コード生成:
      「divisor(Otos_i)が 0 かどうかを確認する.
        0 の場合は ArithmeticException (See: [here](no3059cDE.html) for details)」
      ---------------------------------------- -}

	  __ tst(Otos_i);
	  __ throw_if_not_icc( Assembler::notZero, Interpreter::_throw_ArithmeticException_entry, G3_scratch );
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const int min_int = 0x80000000;
	  Label regular;

  {- -------------------------------------------
  (1) コード生成:
      「divisor(Otos_i)が -1 かどうかを確認する (JVM 仕様上の境界条件のチェック).
        -1 の場合は, このままフォールスルーして境界条件チェックを続ける.
        -1 でない場合は, regular ラベルまでジャンプして除算を行う.」
      ---------------------------------------- -}

	  __ cmp(Otos_i, -1);
	  __ br(Assembler::notEqual, false, Assembler::pt, regular);

  {- -------------------------------------------
  (1) コード生成:
      「divisor(Otos_i)が -1 の場合は, 
        dividend(O1)が int min に等しいかどうかを調べる (等しければ仕様上の境界パターン).
        等しい場合には, dividend(O1)を Otos_i にコピーするだけにして done ラベルにジャンプ.
        そうでなければ, このままフォールスルーして除算処理を行う.                          」
  
       (なお, 64bit の場合には set() は複数の命令を生成するので
       遅延スロットに入れないように注意, とのこと) 
      ---------------------------------------- -}

	#ifdef _LP64
	  // Don't put set in delay slot
	  // Set will turn into multiple instructions in 64 bit mode
	  __ delayed()->nop();
	  __ set(min_int, G4_scratch);
	#else
	  __ delayed()->set(min_int, G4_scratch);
	#endif
	  Label done;
	  __ cmp(O1, G4_scratch);
	  __ br(Assembler::equal, true, Assembler::pt, done);
	  __ delayed()->mov(O1, Otos_i);   // (mov only executed if branch taken)
	
  {- -------------------------------------------
  (1) コード生成:
      「除算の計算を行う」
      ---------------------------------------- -}

	  __ bind(regular);
	  __ sdiv(O1, Otos_i, Otos_i); // note: irem uses O1 after this instruction!

  {- -------------------------------------------
  (1) コード生成:
      「(どの場合も, ここで合流して終了)」
      ---------------------------------------- -}

	  __ bind(done);
	}
	
```


