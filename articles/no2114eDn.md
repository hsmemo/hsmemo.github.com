---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp

### 名前(function name)
```
void ThreadSnapshot::dump_stack_at_safepoint(int max_depth, bool with_locked_monitors) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい ThreadStackTrace オブジェクトを作り, 
      ThreadStackTrace::dump_stack_at_safepoint() でスタックトレースを取得する.
      ---------------------------------------- -}

	  _stack_trace = new ThreadStackTrace(_thread, with_locked_monitors);
	  _stack_trace->dump_stack_at_safepoint(max_depth);
	}
	
```


