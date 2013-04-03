---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp
### 説明(description)

```
// ENTRY routines may lock, GC and throw exceptions

```

### 名前(function name)
```
#define __ENTRY(result_type, header, thread)                         \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  TRACE_CALL(result_type, header)                                    \

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  HandleMarkCleaner __hm(thread);                                    \
	  Thread* THREAD = thread;                                           \

  {- -------------------------------------------
  (1) (以降で, 実際の関数の処理が行われる)
      ---------------------------------------- -}

	  /* begin of body */
	
```


