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
jvmtiCapabilities JvmtiManageCapabilities::init_onload_capabilities() {
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

	#ifndef CC_INTERP
	  jc.can_pop_frame = 1;
	  jc.can_force_early_return = 1;
	#endif // !CC_INTERP
	  jc.can_get_source_debug_extension = 1;
	  jc.can_access_local_variables = 1;
	  jc.can_maintain_original_method_order = 1;
	  jc.can_generate_all_class_hook_events = 1;
	  jc.can_generate_single_step_events = 1;
	  jc.can_generate_exception_events = 1;
	  jc.can_generate_frame_pop_events = 1;
	  jc.can_generate_method_entry_events = 1;
	  jc.can_generate_method_exit_events = 1;
	  jc.can_get_owned_monitor_info = 1;
	  jc.can_get_owned_monitor_stack_depth_info = 1;
	  jc.can_get_current_contended_monitor = 1;
	  // jc.can_get_monitor_info = 1;
	  jc.can_tag_objects = 1;                 // TODO: this should have been removed
	  jc.can_generate_object_free_events = 1; // TODO: this should have been removed
	  return jc;
	}
	
```


