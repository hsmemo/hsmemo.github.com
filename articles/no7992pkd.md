---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvBase.cpp

### 名前(function name)
```
void
JvmtiEnvBase::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Threads::number_of_threads() == 0 || JvmtiThreadState_lock->is_locked(), "sanity check");
	
  {- -------------------------------------------
  (1) この JvmtiEnvBase オブジェクトを
      JvmtiEnvBase::_head_environment フィールドのリストにつないでおく
      (これは生成された全ての JvmtiEnv オブジェクトをつないでおくリスト)
      ---------------------------------------- -}

	  // Add this environment to the end of the environment list (order is important)
	  {
	    // This block of code must not contain any safepoints, as list deallocation
	    // (which occurs at a safepoint) cannot occur simultaneously with this list
	    // addition.  Note: No_Safepoint_Verifier cannot, currently, be used before
	    // threads exist.
	    JvmtiEnvIterator it;
	    JvmtiEnvBase *previous_env = NULL;
	    for (JvmtiEnvBase* env = it.first(); env != NULL; env = it.next(env)) {
	      previous_env = env;
	    }
	    if (previous_env == NULL) {
	      _head_environment = this;
	    } else {
	      previous_env->set_next_environment(this);
	    }
	  }
	
  {- -------------------------------------------
  (1) まだ大域的な初期化が終わっていなければ
      JvmtiEnvBase::globally_initialize() で実行しておく.
      ---------------------------------------- -}

	  if (_globally_initialized == false) {
	    globally_initialize();
	  }
	}
	
```


