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
  (1) JvmtiEnvBase::get_current_contended_monitor() を呼び出すだけ.
      ---------------------------------------- -}

	    _result = ((JvmtiEnvBase *)_env)->get_current_contended_monitor(_calling_thread,_java_thread,_owned_monitor_ptr);
	  }
	
```


