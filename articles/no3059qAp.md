---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.cpp

### 名前(function name)
```
int MacroAssembler::corrected_idivq(Register reg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (JVM の除算/剰余算は, 境界値での挙動が x86 とは異なるため, チェックが必要)
      ---------------------------------------- -}

	  // Full implementation of Java ldiv and lrem; checks for special
	  // case as described in JVM spec., p.243 & p.271.  The function
	  // returns the (pc) offset of the idivl instruction - may be needed
	  // for implicit exceptions.
	  //
	  //         normal case                           special case
	  //
	  // input : rax: dividend                         min_long
	  //         reg: divisor   (may not be eax/edx)   -1
	  //
	  // output: rax: quotient  (= rax idiv reg)       min_long
	  //         rdx: remainder (= rax irem reg)       0

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(reg != rax && reg != rdx, "reg cannot be rax or rdx register");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  static const int64_t min_long = 0x8000000000000000;
	  Label normal_case, special_case;
	
  {- -------------------------------------------
  (1) コード生成:
      「special case かどうかのチェックを行う.
        結果に応じて, 以下の処理は２通りに分岐.」
      ---------------------------------------- -}

	  // check for special case
	  cmp64(rax, ExternalAddress((address) &min_long));
	  jcc(Assembler::notEqual, normal_case);

  {- -------------------------------------------
  (1) コード生成:
      「special case 用の処理を行う.」
      ---------------------------------------- -}

	  xorl(rdx, rdx); // prepare rdx for possible special case (where
	                  // remainder = 0)
	  cmpq(reg, -1);
	  jcc(Assembler::equal, special_case);
	
  {- -------------------------------------------
  (1) コード生成:
      「normal case 用の処理を行う.」
      ---------------------------------------- -}

	  // handle normal case
	  bind(normal_case);
	  cdqq();
	  int idivq_offset = offset();
	  idivq(reg);
	
  {- -------------------------------------------
  (1) コード生成:
      「(special case/normal case どちらもここで合流)」
      ---------------------------------------- -}

	  // normal and special case exit
	  bind(special_case);
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return idivq_offset;
	}
	
```


