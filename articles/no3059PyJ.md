---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/assembler_sparc.cpp

### 名前(function name)
```
void MacroAssembler::null_check(Register reg, int offset) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) MacroAssembler::needs_explicit_null_check() の返り値に応じて, 
      以下のどちらかの挙動を行う.
      * needs_explicit_null_check() が true の場合: 
        明示的にポインタの参照先をロードする. 
        (NULL の場合には, この場で SIGSEGV を起こす)
      * needs_explicit_null_check() が false の場合:
        特に何もしない 
        (NULL の場合には, その後に実際に使用した箇所で SIGSEGV になるまで放置).
      ---------------------------------------- -}

	  if (needs_explicit_null_check((intptr_t)offset)) {
	    // provoke OS NULL exception if reg = NULL by
	    // accessing M[reg] w/o changing any registers
	    ld_ptr(reg, 0, G0);
	  }
	  else {
	    // nothing to do, (later) access of M[reg + offset]
	    // will provoke OS NULL exception if reg = NULL
	  }
	}
	
```


