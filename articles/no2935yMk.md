---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiCodeBlobEvents.cpp
### 説明(description)

```
// Generate a COMPILED_METHOD_LOAD event for each nnmethod
```

### 名前(function name)
```
jvmtiError JvmtiCodeBlobEvents::generate_compiled_method_load_events(JvmtiEnv* env) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  HandleMark hm;
	
  {- -------------------------------------------
  (1) 全ての nmethod オブジェクトに対して 
      nmethod::get_and_cache_jmethod_id() 及び
      JvmtiExport::post_compiled_method_load() を呼び出し, 
      COMPILED_METHOD_LOAD イベントの生成を行う.
    
      (なお, 既に死んでいる nmethod (= is_alive() が false のもの) については生成しない)
  
      (なお, この処理は CodeCache_lock で排他した状態で行う.
       ただし, nmethod::get_and_cache_jmethod_id() と JvmtiExport::post_compiled_method_load() を
       呼び出す瞬間だけはロックを開放して処理する)
      ---------------------------------------- -}

	  // Walk the CodeCache notifying for live nmethods.  The code cache
	  // may be changing while this is happening which is ok since newly
	  // created nmethod will notify normally and nmethods which are freed
	  // can be safely skipped.
	  MutexLockerEx mu(CodeCache_lock, Mutex::_no_safepoint_check_flag);
	  nmethod* current = CodeCache::first_nmethod();
	  while (current != NULL) {
	    // Only notify for live nmethods
	    if (current->is_alive()) {
	      // Lock the nmethod so it can't be freed
	      nmethodLocker nml(current);
	
	      // Don't hold the lock over the notify or jmethodID creation
	      MutexUnlockerEx mu(CodeCache_lock, Mutex::_no_safepoint_check_flag);
	      current->get_and_cache_jmethod_id();
	      JvmtiExport::post_compiled_method_load(current);
	    }
	    current = CodeCache::next_nmethod(current);
	  }

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	}
	
```


