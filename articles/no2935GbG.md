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
void JvmtiManageCapabilities::get_potential_capabilities(const jvmtiCapabilities *current,
                                                         const jvmtiCapabilities *prohibited,
                                                         jvmtiCapabilities *result) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) always_capabilities をベースとして, まず prohibited なものを除外する.
      ---------------------------------------- -}

	  // exclude prohibited capabilities, must be before adding current
	  exclude(&always_capabilities, prohibited, result);
	
  {- -------------------------------------------
  (1) 次に, 現在保持している権限を加える.
      ---------------------------------------- -}

	  // must include current since it may possess solo capabilities and now prohibited
	  either(result, current, result);
	
  {- -------------------------------------------
  (1) 次に, always_solo でまだ残っているものも加える.
      ---------------------------------------- -}

	  // add other remaining
	  either(result, &always_solo_remaining_capabilities, result);
	
  {- -------------------------------------------
  (1) もし OnLoad phase であれば, さらに onload_capabilities と onload_solo_remaining_capabilities を加える.
      ---------------------------------------- -}

	  // if this is during OnLoad more capabilities are available
	  if (JvmtiEnv::get_phase() == JVMTI_PHASE_ONLOAD) {
	    either(result, &onload_capabilities, result);
	    either(result, &onload_solo_remaining_capabilities, result);
	  }
	}
	
```


