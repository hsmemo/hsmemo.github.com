---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.hpp
### 説明(description)

```
  // Serializes memory and blows flags
```

### 名前(function name)
```
  void membar(Membar_mask_bits order_constraint) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) MP 環境でなければ, メモリバリアは必要ないので何もしない.
      ---------------------------------------- -}

	    if (os::is_MP()) {

  {- -------------------------------------------
  (1) 要求されているのが StoreLoad でなければ, バリアは不要なので何もしない.
      ---------------------------------------- -}

	      // We only have to handle StoreLoad
	      if (order_constraint & StoreLoad) {

  {- -------------------------------------------
  (1) lock 付きの nop (より正確には addl [rsp],0) を出すことでメモリバリアコードとする.
      ---------------------------------------- -}

	        // All usable chips support "locked" instructions which suffice
	        // as barriers, and are much faster than the alternative of
	        // using cpuid instruction. We use here a locked add [esp],0.
	        // This is conveniently otherwise a no-op except for blowing
	        // flags.
	        // Any change to this code may need to revisit other places in
	        // the code where this idiom is used, in particular the
	        // orderAccess code.
	        lock();
	        addl(Address(rsp, 0), 0);// Assert the lock# signal here
	      }
	    }
	  }
	
```


