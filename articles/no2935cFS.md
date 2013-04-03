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
static inline void generate_satb_log_enqueue_if_necessary(bool with_frame) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 既に satb_log_enqueue_with_frame (with_frame 引数が false の場合は satb_log_enqueue_frameless) が
      初期化済みであれば何もしない.
  
      まだ初期化されてなければ, generate_satb_log_enqueue() を呼んで初期化を行う.
  
      (なお, 初期化を行った場合はついでに(トレース出力)も出している)
      ---------------------------------------- -}

	  if (with_frame) {
	    if (satb_log_enqueue_with_frame == 0) {
	      generate_satb_log_enqueue(with_frame);
	      assert(satb_log_enqueue_with_frame != 0, "postcondition.");
	      if (G1SATBPrintStubs) {
	        tty->print_cr("Generated with-frame satb enqueue:");
	        Disassembler::decode((u_char*)satb_log_enqueue_with_frame,
	                             satb_log_enqueue_with_frame_end,
	                             tty);
	      }
	    }
	  } else {
	    if (satb_log_enqueue_frameless == 0) {
	      generate_satb_log_enqueue(with_frame);
	      assert(satb_log_enqueue_frameless != 0, "postcondition.");
	      if (G1SATBPrintStubs) {
	        tty->print_cr("Generated frameless satb enqueue:");
	        Disassembler::decode((u_char*)satb_log_enqueue_frameless,
	                             satb_log_enqueue_frameless_end,
	                             tty);
	      }
	    }
	  }
	}
	
```


