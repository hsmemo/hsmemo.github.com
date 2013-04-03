---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/assembler_sparc.inline.hpp

### 名前(function name)
```
inline void MacroAssembler::store_double_argument( FloatRegister s, Argument& a ) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      引数で指定されたコピー先に応じて以下のどちらかのコードを生成.
      * store 先がレジスタの場合:
        「fmov 命令でコピーする.」
      * そうではない場合:
        「stf でストアする.」
      ---------------------------------------- -}

	  if (a.is_float_register())
	// V9 ABI has D0, D2, D4 are used to pass instead of O0, O1, O2
	    fmov(FloatRegisterImpl::D, s, a.as_double_register() );
	  else
	    stf(FloatRegisterImpl::D, s, a.as_address());
	}
	
```


