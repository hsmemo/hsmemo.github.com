---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateTable_x86_64.cpp
### 説明(description)

```
// ----------------------------------------------------------------------------
// Volatile variables demand their effects be made known to all CPU's
// in order.  Store buffers on most chips allow reads & writes to
// reorder; the JMM's ReadAfterWrite.java test fails in -Xint mode
// without some kind of memory barrier (i.e., it's not sufficient that
// the interpreter does not reorder volatile references, the hardware
// also must not reorder them).
//
// According to the new Java Memory Model (JMM):
// (1) All volatiles are serialized wrt to each other.  ALSO reads &
//     writes act as aquire & release, so:
// (2) A read cannot let unrelated NON-volatile memory refs that
//     happen after the read float up to before the read.  It's OK for
//     non-volatile memory refs that happen before the volatile read to
//     float down below it.
// (3) Similar a volatile write cannot let unrelated NON-volatile
//     memory refs that happen BEFORE the write float down to after the
//     write.  It's OK for non-volatile memory refs that happen after the
//     volatile write to float up before it.
//
// We only put in barriers around volatile refs (they are expensive),
// not _between_ memory refs (that would require us to track the
// flavor of the previous memory refs).  Requirements (2) and (3)
// require some barriers before volatile stores and after volatile
// loads.  These nearly cover requirement (1) but miss the
// volatile-store-volatile-load case.  This final case is placed after
// volatile-stores although it could just as well go before
// volatile-loads.
```

### 名前(function name)
```
void TemplateTable::volatile_barrier(Assembler::Membar_mask_bits
                                     order_constraint) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) MP 環境でなければ (os::is_MP() が true を返さなければ) 何もしない.
      層でなければ, MacroAssembler::membar() でメモリバリアコードを生成.
      ---------------------------------------- -}

	  // Helper function to insert a is-volatile test and memory barrier
	  if (os::is_MP()) { // Not needed on single CPU
	    __ membar(order_constraint);
	  }
	}
	
```


