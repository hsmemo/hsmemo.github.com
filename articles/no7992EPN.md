---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiManageCapabilities.cpp
### 説明(description)

```
// if the capability sets are initialized in the onload phase then
// it happens before class data sharing (CDS) is initialized. If it
// turns out that CDS gets disabled then we must adjust the always
// capabilities. To ensure a consistent view of the capabililties
// anything we add here should already be in the onload set.
```

### 名前(function name)
```
void JvmtiManageCapabilities::recompute_always_capabilities() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (フィールドの初期化)
      (always_capabilities フィールドの can_generate_all_class_hook_events を初期化)
      ---------------------------------------- -}

	  if (!UseSharedSpaces) {
	    jvmtiCapabilities jc = always_capabilities;
	    jc.can_generate_all_class_hook_events = 1;
	    always_capabilities = jc;
	  }
	}
	
```


