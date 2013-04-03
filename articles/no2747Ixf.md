---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/sharedRuntime.cpp
### 説明(description)
(なぜか #ifndef PRODUCT が二重にかかっているが... #TODO)

```
#ifndef PRODUCT
```

### 名前(function name)
```
JRT_ENTRY(intptr_t, SharedRuntime::trace_bytecode(JavaThread* thread, intptr_t preserve_this_value, intptr_t tos, intptr_t tos2))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const frame f = thread->last_frame();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(f.is_interpreted_frame(), "must be an interpreted frame");

  {- -------------------------------------------
  (1) BytecodeTracer::trace() を呼び出すだけ.
      ---------------------------------------- -}

	#ifndef PRODUCT
	  methodHandle mh(THREAD, f.interpreter_frame_method());
	  BytecodeTracer::trace(mh, f.interpreter_frame_bcp(), tos, tos2);
	#endif // !PRODUCT
	
```


