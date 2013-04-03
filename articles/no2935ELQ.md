---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvBase.hpp

### 名前(function name)
```
  void doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiEnvBase::get_frame_count() を呼び出すだけ.
      ---------------------------------------- -}

	    _result = ((JvmtiEnvBase*)_env)->get_frame_count(_state, _count_ptr);
	  }
	
```


