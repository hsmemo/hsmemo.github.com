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
// LEAF routines do not lock, GC or throw exceptions

```

### 名前(function name)
```
#define __LEAF(result_type, header)                                  \
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

	  debug_only(NoHandleMark __hm;)                                     \

  {- -------------------------------------------
  (1) (以降で, 実際の関数の処理が行われる)
      ---------------------------------------- -}

	  /* begin of body */
	
```


