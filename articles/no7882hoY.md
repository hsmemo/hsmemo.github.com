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
// Same as JRT Entry but allows for return value after the safepoint
// to get back into Java from the VM
```

### 名前(function name)
```
#define JRT_BLOCK_ENTRY(result_type, header)                         \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (関数定義の関数名および型宣言部が生成される)
      ---------------------------------------- -}

	  result_type header {                                               \

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    TRACE_CALL(result_type, header)                                  \

  {- -------------------------------------------
  (1) (変数宣言など) 
      (See: HandleMarkCleaner)
      ---------------------------------------- -}

	    HandleMarkCleaner __hm(thread);
	
```


