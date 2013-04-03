---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiManageCapabilities.cpp

### 名前(function name)
```
jvmtiCapabilities JvmtiManageCapabilities::init_onload_solo_capabilities() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      (jc は, リターンする結果を入れる変数)
      ---------------------------------------- -}

	  jvmtiCapabilities jc;
	
  {- -------------------------------------------
  (1) 最初に jc を 0 クリアしておく.
      ---------------------------------------- -}

	  memset(&jc, 0, sizeof(jc));

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  jc.can_generate_field_modification_events = 1;
	  jc.can_generate_field_access_events = 1;
	  jc.can_generate_breakpoint_events = 1;
	  return jc;
	}
	
```


