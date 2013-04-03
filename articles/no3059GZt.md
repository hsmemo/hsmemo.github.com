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
void MacroAssembler::c2bool(Register x) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「andl で, 対象のレジスタ(x)の下位 1 byte が 0 かどうかを確認し, 
        setb で, 非ゼロなら 1, ゼロなら 0 をセットする.」
    
       (なおコメントによると, andl でわざわざ下位 1 byte だけに限定してみているのは
        C-style boolean はそこにしか書き込まれないため, とのこと)
      ---------------------------------------- -}

	  // implements x == 0 ? 0 : 1
	  // note: must only look at least-significant byte of x
	  //       since C-style booleans are stored in one byte
	  //       only! (was bug)
	  andl(x, 0xFF);
	  setb(Assembler::notZero, x);
	}
	
```


