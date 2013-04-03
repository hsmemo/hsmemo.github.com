---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/jniHandles.cpp

### 名前(function name)
```
bool JNIHandles::is_weak_global_handle(jobject handle) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) handle 引数で指定されたオブジェクトが
      _weak_global_handles 内にあれば true をリターン.
      (逆に, なければ false をリターン)
      ---------------------------------------- -}

	  return _weak_global_handles->chain_contains(handle);
	}
	
```


