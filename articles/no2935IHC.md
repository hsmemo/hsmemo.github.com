---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/register_x86.hpp
### 説明(description)

```
// The implementation of integer registers for the ia32 architecture
```

### 名前(function name)
```
inline Register as_Register(int encoding) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) encoding 引数の値をそのままリターンするだけ.
      ---------------------------------------- -}

	  return (Register)(intptr_t) encoding;
	}
	
```


