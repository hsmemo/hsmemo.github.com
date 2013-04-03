---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/utilities/debug.cpp

### 名前(function name)
```
void report_java_out_of_memory(const char* message) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (なお, この処理が複数回実行されないよう, out_of_memory_reported という変数で排他を取っている)
      ---------------------------------------- -}

	  static jint out_of_memory_reported = 0;
	
	  // A number of threads may attempt to report OutOfMemoryError at around the
	  // same time. To avoid dumping the heap or executing the data collection
	  // commands multiple times we just do it once when the first threads reports
	  // the error.
	  if (Atomic::cmpxchg(1, &out_of_memory_reported, 0) == 0) {

  {- -------------------------------------------
  (1) HeapDumpOnOutOfMemoryError オプションが指定されていれば, HeapDumper にヒープをダンプさせる.
      ---------------------------------------- -}

	    // create heap dump before OnOutOfMemoryError commands are executed
	    if (HeapDumpOnOutOfMemoryError) {
	      tty->print_cr("java.lang.OutOfMemoryError: %s", message);
	      HeapDumper::dump_heap_from_oome();
	    }
	
  {- -------------------------------------------
  (1) OnOutOfMemoryError オプションが指定されていれば, VMError::report_java_out_of_memory() を呼び出してユーザー指定のコマンドを起動させる.
      ---------------------------------------- -}

	    if (OnOutOfMemoryError && OnOutOfMemoryError[0]) {
	      VMError err(message);
	      err.report_java_out_of_memory();
	    }
	  }
	}
	
```


