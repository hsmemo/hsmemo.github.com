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
static void pass_arg3(MacroAssembler* masm, Register arg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) c_rarg3 レジスタにコピーするだけ
      ---------------------------------------- -}

	  if (c_rarg3 != arg ) {
	    masm->mov(c_rarg3, arg);
	  }
	}
	
```


