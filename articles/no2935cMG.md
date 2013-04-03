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
static inline void
generate_dirty_card_log_enqueue_if_necessary(jbyte* byte_map_base) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 既に dirty_card_log_enqueue が初期化済みであれば何もしない.
  
      まだ初期化されてなければ, generate_dirty_card_log_enqueue() を呼んで初期化を行う.
  
      (なお, 初期化を行った場合はついでに(トレース出力)も出している)
      ---------------------------------------- -}

	  if (dirty_card_log_enqueue == 0) {
	    generate_dirty_card_log_enqueue(byte_map_base);
	    assert(dirty_card_log_enqueue != 0, "postcondition.");
	    if (G1SATBPrintStubs) {
	      tty->print_cr("Generated dirty_card enqueue:");
	      Disassembler::decode((u_char*)dirty_card_log_enqueue,
	                           dirty_card_log_enqueue_end,
	                           tty);
	    }
	  }
	}
	
```


