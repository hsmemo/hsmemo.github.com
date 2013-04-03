---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvBase.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) _prohibited_capabilities フィールドのアドレスをリターンするだけ.
      ---------------------------------------- -}

	  jvmtiCapabilities *get_prohibited_capabilities()  { return &_prohibited_capabilities; }
	
```


