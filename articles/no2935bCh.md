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
// corresponding init functions
```

### 名前(function name)
```
jvmtiCapabilities JvmtiManageCapabilities::init_always_capabilities() {
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

	  jc.can_get_bytecodes = 1;
	  jc.can_signal_thread = 1;
	  jc.can_get_source_file_name = 1;
	  jc.can_get_line_numbers = 1;
	  jc.can_get_synthetic_attribute = 1;
	  jc.can_get_monitor_info = 1;
	  jc.can_get_constant_pool = 1;
	  jc.can_generate_monitor_events = 1;
	  jc.can_generate_garbage_collection_events = 1;
	  jc.can_generate_compiled_method_load_events = 1;
	  jc.can_generate_native_method_bind_events = 1;
	  jc.can_generate_vm_object_alloc_events = 1;
	  if (os::is_thread_cpu_time_supported()) {
	    jc.can_get_current_thread_cpu_time = 1;
	    jc.can_get_thread_cpu_time = 1;
	  }
	  jc.can_redefine_classes = 1;
	  jc.can_redefine_any_class = 1;
	  jc.can_retransform_classes = 1;
	  jc.can_retransform_any_class = 1;
	  jc.can_set_native_method_prefix = 1;
	  jc.can_tag_objects = 1;
	  jc.can_generate_object_free_events = 1;
	  jc.can_generate_resource_exhaustion_heap_events = 1;
	  jc.can_generate_resource_exhaustion_threads_events = 1;
	  return jc;
	}
	
```


