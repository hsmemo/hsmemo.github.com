---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiExport.cpp
### 説明(description)

```
// Setup current current thread for event collection.
```

### 名前(function name)
```
void JvmtiEventCollector::setup_jvmti_thread_state() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiThreadState::state_for() を呼んで, 
      カレントスレッドに対応する JvmtiThreadState を取得する.
      (もし対象のスレッドが JvmtiThreadState を持っていなければ, この中で確保処理が行われる)
      ---------------------------------------- -}

	  // set this event collector to be the current one.
	  JvmtiThreadState* state = JvmtiThreadState::state_for(JavaThread::current());

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // state can only be NULL if the current thread is exiting which
	  // should not happen since we're trying to configure for event collection
	  guarantee(state != NULL, "exiting thread called setup_jvmti_thread_state");

  {- -------------------------------------------
  (1) この JvmtiEventCollector (のサブクラスの) オブジェクトを
      JvmtiThreadState オブジェクト内の適切なリストに追加する.
      (ついでに, 変更前の値を _prev フィールドに保存しておくことで線形リストを構築)
      
      * JvmtiVMObjectAllocEventCollector オブジェクトの場合:
        JvmtiThreadState::set_vm_object_alloc_event_collector() を呼び出して追加.
      * JvmtiDynamicCodeEventCollector オブジェクトの場合:
        JvmtiThreadState::set_dynamic_code_event_collector() を呼び出して追加.
      ---------------------------------------- -}

	  if (is_vm_object_alloc_event()) {
	    _prev = state->get_vm_object_alloc_event_collector();
	    state->set_vm_object_alloc_event_collector((JvmtiVMObjectAllocEventCollector *)this);
	  } else if (is_dynamic_code_event()) {
	    _prev = state->get_dynamic_code_event_collector();
	    state->set_dynamic_code_event_collector((JvmtiDynamicCodeEventCollector *)this);
	  }
	}
	
```


