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
// Close the routine and the extern "C"
```


### 本体部(body)
```
  {- -------------------------------------------
  (1) (単に "} }" に展開されるだけ. 
      ただし, ここで様々なデストラクタが走るため, 処理がないわけではない)
      ---------------------------------------- -}

	#define JNI_END } }
	
```


