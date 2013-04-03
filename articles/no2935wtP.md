---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/nativeInst_x86.hpp

### 名前(function name)
```
inline NativeGeneralJump* nativeGeneralJump_at(address address) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) address 引数の値をそのままリターンするだけ.
      ---------------------------------------- -}

	  NativeGeneralJump* jump = (NativeGeneralJump*)(address);
	  debug_only(jump->verify();)
	  return jump;
	}
	
```


