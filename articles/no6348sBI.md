---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/compilationPolicy.cpp

### 名前(function name)
```
void SimpleCompPolicy::method_invocation_event( methodHandle m, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int hot_count = m->invocation_count();

  {- -------------------------------------------
  (1) NonTieredCompPolicy::reset_counter_for_invocation_event() を呼んで
      (すぐにまた JIT コンパイルが発生しないように) InvocationCounter を下げておく.
      ---------------------------------------- -}

	  reset_counter_for_invocation_event(m);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const char* comment = "count";
	
  {- -------------------------------------------
  (1) CompileBroker::compile_method() を呼んで, JIT コンパイル処理を行う.
      
      ただし, 以下の場合は JIT コンパイル処理は行わない.
      * JIT コンパイル処理を行ってはいけない状況 (= CompilationPolicy::is_compilation_enabled() が false) の場合
      * 処理対象が, JIT コンパイルが禁止されているメソッドである場合 (= CompilationPolicy::can_be_compiled() が false)
      * (一足違いで先に?) 並行に動いている JIT コンパイラスレッドによって JIT コンパイルが完了されてしまった場合
      ---------------------------------------- -}

	  if (is_compilation_enabled() && can_be_compiled(m)) {
	    nmethod* nm = m->code();
	    if (nm == NULL ) {
	      const char* comment = "count";
	      CompileBroker::compile_method(m, InvocationEntryBci, CompLevel_highest_tier,
	                                    m, hot_count, comment, CHECK);
	    }
	  }
	}
	
```


