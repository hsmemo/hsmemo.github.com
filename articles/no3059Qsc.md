---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateTable_x86_64.cpp

### 名前(function name)
```
void TemplateTable::ldiv() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(ltos, ltos);

  {- -------------------------------------------
  (1) コード生成:
      「divosor を rcx レジスタに待避し, 
        dividend を rxa レジスタにロードする.」
      ---------------------------------------- -}

	  __ mov(rcx, rax);
	  __ pop_l(rax);

  {- -------------------------------------------
  (1) コード生成:
      「divisor(rcx)が 0 かどうかを確認する.
        0 の場合は ArithmeticException (See: [here](no3059cDE.html) for details).
        そうでなければ, corrected_idivq() が生成するコードにより
        除算計算を実施.」
  
      (なおコメントとして, 最適化のアイデアが書かれていたりする)
      ---------------------------------------- -}

	  // generate explicit div0 check
	  __ testq(rcx, rcx);
	  __ jump_cc(Assembler::zero,
	             ExternalAddress(Interpreter::_throw_ArithmeticException_entry));
	  // Note: could xor rax and rcx and compare with (-1 ^ min_int). If
	  //       they are not equal, one could do a normal division (no correction
	  //       needed), which may speed up this implementation for the common case.
	  //       (see also JVM spec., p.243 & p.271)
	  __ corrected_idivq(rcx); // kills rbx
	}
	
```


