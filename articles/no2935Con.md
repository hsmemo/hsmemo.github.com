---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp

### 名前(function name)
```
jvmtiError
JvmtiEnv::ClearFieldModificationWatch(fieldDescriptor* fdesc_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	   // make sure we have a watch to clear
	  if (!fdesc_ptr->is_field_modification_watched()) return JVMTI_ERROR_NOT_FOUND;
	  fdesc_ptr->set_is_field_modification_watched(false);
	  update_klass_field_access_flag(fdesc_ptr);
	
	  JvmtiEventController::change_field_watch(JVMTI_EVENT_FIELD_MODIFICATION, false);
	
	  return JVMTI_ERROR_NONE;
	} /* end ClearFieldModificationWatch */
	
```


