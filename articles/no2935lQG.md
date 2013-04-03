---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEventController.cpp
### 説明(description)

```
// Compute truly enabled events - meaning if the event can and could be
// sent.  An event is truly enabled if it is user enabled on the thread
// or globally user enabled, but only if there is a callback or event hook
// for it and, for field watch and frame pop, one has been set.
// Compute if truly enabled, per thread, per environment, per combination
// (thread x environment), and overall.  These merges are true if any is true.
// True per thread if some environment has callback set and the event is globally
// enabled or enabled for this thread.
// True per environment if the callback is set and the event is globally
// enabled in this environment or enabled for any thread in this environment.
// True per combination if the environment has the callback set and the
// event is globally enabled in this environment or the event is enabled
// for this thread and environment.
//
// All states transitions dependent on these transitions are also handled here.
```

### 名前(function name)
```
void
JvmtiEventControllerPrivate::recompute_enabled() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Threads::number_of_threads() == 0 || JvmtiThreadState_lock->is_locked(), "sanity check");
	
  {- -------------------------------------------
  (1) 現在の truly enabled event 設定を取得.
      ---------------------------------------- -}

	  // event enabled for any thread in any environment
	  jlong was_any_env_thread_enabled = JvmtiEventController::_universal_global_event_enabled.get_bits();

  {- -------------------------------------------
  (1) (変数宣言など)
      (any_env_thread_enabled は新しい truly enabled event の計算結果を格納する変数)
      ---------------------------------------- -}

	  jlong any_env_thread_enabled = 0;
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EC_TRACE(("JVMTI [-] # recompute enabled - before %llx", was_any_env_thread_enabled));
	
  {- -------------------------------------------
  (1) (以降で, 新しい truly enabled event を計算する)
      ---------------------------------------- -}

	  // compute non-thread-filters events.
	  // This must be done separately from thread-filtered events, since some
	  // events can occur before any threads exist.

    {- -------------------------------------------
  (1.1) 全ての JvmtiEnvBase オブジェクトをたどり, 
        各 JVMTI environment に対して JvmtiEventControllerPrivate::recompute_env_enabled() を呼び出す.
        そして, その結果を any_env_thread_enabled に足し込んでいく.
        ---------------------------------------- -}

	  JvmtiEnvIterator it;
	  for (JvmtiEnvBase* env = it.first(); env != NULL; env = it.next(env)) {
	    any_env_thread_enabled |= recompute_env_enabled(env);
	  }
	
    {- -------------------------------------------
  (1.1) もし, これまではどの thread filtered events (各スレッドごとの処理に同期して発生する event) も有効ではなかったのに
        今回の更新で thread filtered events が1つでも有効になっていたら,
        各 JavaThread に対して (まだ作られていなければ) JvmtiThreadState を作成する処理を行う.
  
        (具体的にどれが thread filtered events かは THREAD_FILTERED_EVENT_BITS の定義を参照.
        なお, 非 thread filtered event だけを集めた GLOBAL_EVENT_BITS という bit mask もある.
  
        参考: 非 thread filtered event の例
          * THREAD_START_BIT,
          * CLASS_FILE_LOAD_HOOK_BIT
          * VM_START_BIT, VM_INIT_BIT, VM_DEATH_BIT,
          * NATIVE_METHOD_BIND_BIT,
          * DYNAMIC_CODE_GENERATED_BIT                                                )
        ---------------------------------------- -}

	  // We need to create any missing jvmti_thread_state if there are globally set thread
	  // filtered events and there weren't last time
	  if (    (any_env_thread_enabled & THREAD_FILTERED_EVENT_BITS) != 0 &&
	      (was_any_env_thread_enabled & THREAD_FILTERED_EVENT_BITS) == 0) {
	    assert(JvmtiEnv::is_vm_live() || (JvmtiEnv::get_phase()==JVMTI_PHASE_START),
	      "thread filtered events should not be enabled when VM not in start or live phase");
	    {
	      MutexLocker mu(Threads_lock);   //hold the Threads_lock for the iteration
	      for (JavaThread *tp = Threads::first(); tp != NULL; tp = tp->next()) {
	        // state_for_while_locked() makes tp->is_exiting() check
	        JvmtiThreadState::state_for_while_locked(tp);  // create the thread state if missing
	      }
	    }// release Threads_lock
	  }
	
    {- -------------------------------------------
  (1.1) 全ての JvmtiThreadState をたどり, JvmtiEventControllerPrivate::recompute_thread_enabled() を呼び出す.
        そして, その結果を any_env_thread_enabled に足し込んでいく.
        ---------------------------------------- -}

	  // compute and set thread-filtered events
	  for (JvmtiThreadState *state = JvmtiThreadState::first(); state != NULL; state = state->next()) {
	    any_env_thread_enabled |= recompute_thread_enabled(state);
	  }
	
  {- -------------------------------------------
  (1) もし新しい truly enabled event が古いものと異なれば, 
      以下の if ブロック内でその変更を反映させる処理を行う.
      (行う処理は主に JvmtiExport::set_should_post_*() を呼び出すことのみ.
       なお, JvmtiExport::set_should_post_*() はソースコード上での定義位置が分かりにくいので注意 (See: JvmtiExport))
      ---------------------------------------- -}

	  // set universal state (across all envs and threads)
	  jlong delta = any_env_thread_enabled ^ was_any_env_thread_enabled;
	  if (delta != 0) {

    {- -------------------------------------------
  (1.1) JvmtiExport::set_should_post_*() を呼び出す
        ---------------------------------------- -}

	    JvmtiExport::set_should_post_field_access((any_env_thread_enabled & FIELD_ACCESS_BIT) != 0);
	    JvmtiExport::set_should_post_field_modification((any_env_thread_enabled & FIELD_MODIFICATION_BIT) != 0);
	    JvmtiExport::set_should_post_class_load((any_env_thread_enabled & CLASS_LOAD_BIT) != 0);
	    JvmtiExport::set_should_post_class_file_load_hook((any_env_thread_enabled & CLASS_FILE_LOAD_HOOK_BIT) != 0);
	    JvmtiExport::set_should_post_native_method_bind((any_env_thread_enabled & NATIVE_METHOD_BIND_BIT) != 0);
	    JvmtiExport::set_should_post_dynamic_code_generated((any_env_thread_enabled & DYNAMIC_CODE_GENERATED_BIT) != 0);
	    JvmtiExport::set_should_post_data_dump((any_env_thread_enabled & DATA_DUMP_BIT) != 0);
	    JvmtiExport::set_should_post_class_prepare((any_env_thread_enabled & CLASS_PREPARE_BIT) != 0);
	    JvmtiExport::set_should_post_class_unload((any_env_thread_enabled & CLASS_UNLOAD_BIT) != 0);
	    JvmtiExport::set_should_post_monitor_contended_enter((any_env_thread_enabled & MONITOR_CONTENDED_ENTER_BIT) != 0);
	    JvmtiExport::set_should_post_monitor_contended_entered((any_env_thread_enabled & MONITOR_CONTENDED_ENTERED_BIT) != 0);
	    JvmtiExport::set_should_post_monitor_wait((any_env_thread_enabled & MONITOR_WAIT_BIT) != 0);
	    JvmtiExport::set_should_post_monitor_waited((any_env_thread_enabled & MONITOR_WAITED_BIT) != 0);
	    JvmtiExport::set_should_post_garbage_collection_start((any_env_thread_enabled & GARBAGE_COLLECTION_START_BIT) != 0);
	    JvmtiExport::set_should_post_garbage_collection_finish((any_env_thread_enabled & GARBAGE_COLLECTION_FINISH_BIT) != 0);
	    JvmtiExport::set_should_post_object_free((any_env_thread_enabled & OBJECT_FREE_BIT) != 0);
	    JvmtiExport::set_should_post_resource_exhausted((any_env_thread_enabled & RESOURCE_EXHAUSTED_BIT) != 0);
	    JvmtiExport::set_should_post_compiled_method_load((any_env_thread_enabled & COMPILED_METHOD_LOAD_BIT) != 0);
	    JvmtiExport::set_should_post_compiled_method_unload((any_env_thread_enabled & COMPILED_METHOD_UNLOAD_BIT) != 0);
	    JvmtiExport::set_should_post_vm_object_alloc((any_env_thread_enabled & VM_OBJECT_ALLOC_BIT) != 0);
	
	    // need this if we want thread events or we need them to init data
	    JvmtiExport::set_should_post_thread_life((any_env_thread_enabled & NEED_THREAD_LIFE_EVENTS) != 0);
	
    {- -------------------------------------------
  (1.1) SINGLE_STEP_BIT に変更が有った場合には, VM_ChangeSingleStep を実行して SingleStep 状態の ON/OFF を行う.
  
        (ただし, この処理は JVMTI_PHASE_LIVE の場合にしか行わない.
         JVMTI_PHASE_DEAD であれば無視するだけ. それ以外の phase であれば assert failure.)
        ---------------------------------------- -}

	    // If single stepping is turned on or off, execute the VM op to change it.
	    if (delta & SINGLE_STEP_BIT) {
	      switch (JvmtiEnv::get_phase()) {
	      case JVMTI_PHASE_DEAD:
	        // If the VM is dying we can't execute VM ops
	        break;
	      case JVMTI_PHASE_LIVE: {
	        VM_ChangeSingleStep op((any_env_thread_enabled & SINGLE_STEP_BIT) != 0);
	        VMThread::execute(&op);
	        break;
	      }
	      default:
	        assert(false, "should never come here before live phase");
	        break;
	      }
	    }
	
    {- -------------------------------------------
  (1.1) 新しい truly enabled event を
        JvmtiEventController::_universal_global_event_enabled にキャッシュしておく.
        ---------------------------------------- -}

	    // set global truly enabled, that is, any thread in any environment
	    JvmtiEventController::_universal_global_event_enabled.set_bits(any_env_thread_enabled);
	
    {- -------------------------------------------
  (1.1) JvmtiExport::set_should_post_*() を呼び出す
        ---------------------------------------- -}

	    // set global should_post_on_exceptions
	    JvmtiExport::set_should_post_on_exceptions((any_env_thread_enabled & SHOULD_POST_ON_EXCEPTIONS_BITS) != 0);
	
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EC_TRACE(("JVMTI [-] # recompute enabled - after %llx", any_env_thread_enabled));
	}
	
```


