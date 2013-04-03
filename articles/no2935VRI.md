---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/register_x86.hpp

### 名前(function name)
```
inline FloatRegister as_FloatRegister(int encoding) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) encoding 引数の値をそのままリターンするだけ.
      ---------------------------------------- -}

	  return (FloatRegister)(intptr_t) encoding;
	}
	
```


