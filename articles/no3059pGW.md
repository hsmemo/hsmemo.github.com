---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/asm/assembler.cpp

### 名前(function name)
```
bool MacroAssembler::needs_explicit_null_check(intptr_t offset) {
```

### 本体部(body)
```
	  // Exception handler checks the nmethod's implicit null checks table
	  // only when this method returns false.

  {- -------------------------------------------
  (1) (64 bit 環境で, UseCompressedOops が使用されており, さらに heap base が設定されている場合には,
       オフセットは heap base からの相対値で考えないといけない.
       offset の値を, それに合わせて修正しておく.)
      (See: UseCompressedOops)
      ---------------------------------------- -}

	#ifdef _LP64
	  if (UseCompressedOops && Universe::narrow_oop_base() != NULL) {
	    assert (Universe::heap() != NULL, "java heap should be initialized");
	    // The first page after heap_base is unmapped and
	    // the 'offset' is equal to [heap_base + offset] for
	    // narrow oop implicit null checks.
	    uintptr_t base = (uintptr_t)Universe::narrow_oop_base();
	    if ((uintptr_t)offset >= base) {
	      // Normalize offset for the next check.
	      offset = (intptr_t)(pointer_delta((void*)offset, (void*)base, 1));
	    }
	  }
	#endif

  {- -------------------------------------------
  (1) 「アクセス先のオフセットが 0 以上かつ pagesize 未満」であれば false をリターン. 
      そうでなければ true をリターン.
      ---------------------------------------- -}

	  return offset < 0 || os::vm_page_size() <= offset;
	}
	
```


