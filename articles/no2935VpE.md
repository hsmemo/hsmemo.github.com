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
// post a DynamicCodeGenerated event while holding locks in the VM.
```

### 名前(function name)
```
void JvmtiExport::post_dynamic_code_generated_while_holding_locks(const char* name,
                                                                  address code_begin, address code_end)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiThreadState::state_for() を呼んで, 
      カレントスレッドに対応する JvmtiThreadState を取得する.
      (もし対象のスレッドが JvmtiThreadState を持っていなければ, この中で確保処理が行われる)
      ---------------------------------------- -}

	  // register the stub with the current dynamic code event collector
	  JvmtiThreadState* state = JvmtiThreadState::state_for(JavaThread::current());

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // state can only be NULL if the current thread is exiting which
	  // should not happen since we're trying to post an event
	  guarantee(state != NULL, "attempt to register stub via an exiting thread");

  {- -------------------------------------------
  (1) JvmtiThreadState::get_dynamic_code_event_collector() を呼んで
      最も直近で生成された JvmtiDynamicCodeEventCollector オブジェクトを取得する.
      ---------------------------------------- -}

	  JvmtiDynamicCodeEventCollector* collector = state->get_dynamic_code_event_collector();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee(collector != NULL, "attempt to register stub without event collector");

  {- -------------------------------------------
  (1) 取得した JvmtiDynamicCodeEventCollector オブジェクトに対して
      JvmtiDynamicCodeEventCollector::register_stub() を呼び出し, 
      引数で指定されたコード情報をその中に記録しておく.
      ---------------------------------------- -}

	  collector->register_stub(name, code_begin, code_end);
	}
	
```


