---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp

### 名前(function name)
```
ThreadSnapshot::ThreadSnapshot(JavaThread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  _thread = thread;
	  _threadObj = thread->threadObj();
	  _stack_trace = NULL;
	  _concurrent_locks = NULL;
	  _next = NULL;
	
	  ThreadStatistics* stat = thread->get_thread_stat();
	  _contended_enter_ticks = stat->contended_enter_ticks();
	  _contended_enter_count = stat->contended_enter_count();
	  _monitor_wait_ticks = stat->monitor_wait_ticks();
	  _monitor_wait_count = stat->monitor_wait_count();
	  _sleep_ticks = stat->sleep_ticks();
	  _sleep_count = stat->sleep_count();
	
	  _blocker_object = NULL;
	  _blocker_object_owner = NULL;
	
	  _thread_status = java_lang_Thread::get_thread_status(_threadObj);
	  _is_ext_suspended = thread->is_being_ext_suspended();
	  _is_in_native = (thread->thread_state() == _thread_in_native);
	
	  if (_thread_status == java_lang_Thread::BLOCKED_ON_MONITOR_ENTER ||
	      _thread_status == java_lang_Thread::IN_OBJECT_WAIT ||
	      _thread_status == java_lang_Thread::IN_OBJECT_WAIT_TIMED) {
	
	    Handle obj = ThreadService::get_current_contended_monitor(thread);
	    if (obj() == NULL) {
	      // monitor no longer exists; thread is not blocked
	      _thread_status = java_lang_Thread::RUNNABLE;
	    } else {
	      _blocker_object = obj();
	      JavaThread* owner = ObjectSynchronizer::get_lock_owner(obj, false);
	      if ((owner == NULL && _thread_status == java_lang_Thread::BLOCKED_ON_MONITOR_ENTER)
	          || (owner != NULL && owner->is_attaching())) {
	        // ownership information of the monitor is not available
	        // (may no longer be owned or releasing to some other thread)
	        // make this thread in RUNNABLE state.
	        // And when the owner thread is in attaching state, the java thread
	        // is not completely initialized. For example thread name and id
	        // and may not be set, so hide the attaching thread.
	        _thread_status = java_lang_Thread::RUNNABLE;
	        _blocker_object = NULL;
	      } else if (owner != NULL) {
	        _blocker_object_owner = owner->threadObj();
	      }
	    }
	  }
	
	  // Support for JSR-166 locks
	  if (JDK_Version::current().supports_thread_park_blocker() &&
	        (_thread_status == java_lang_Thread::PARKED ||
	         _thread_status == java_lang_Thread::PARKED_TIMED)) {
	
	    _blocker_object = thread->current_park_blocker();
	    if (_blocker_object != NULL && _blocker_object->is_a(SystemDictionary::abstract_ownable_synchronizer_klass())) {
	      _blocker_object_owner = java_util_concurrent_locks_AbstractOwnableSynchronizer::get_owner_threadObj(_blocker_object);
	    }
	  }
	}
	
```


