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
JvmtiEnvBase::record_class_file_load_hook_enabled() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) まだ一度も class_file_load_hook が enabled になっていなければ, 
      JvmtiEnvBase::record_first_time_class_file_load_hook_enabled() を呼び出す.
  
      (なお, HotSpot の起動時(= Threads::number_of_threads() が 0)であれば
       ロックは必要ない(というかロック機構自体が立ち上がっていない)のでロックを取らずに呼び出す.
       そうでなければ JvmtiThreadState_lock を確保した状態で呼び出す.)
      ---------------------------------------- -}

	  if (!_class_file_load_hook_ever_enabled) {
	    if (Threads::number_of_threads() == 0) {
	      record_first_time_class_file_load_hook_enabled();
	    } else {
	      MutexLocker mu(JvmtiThreadState_lock);
	      record_first_time_class_file_load_hook_enabled();
	    }
	  }
	}
	
```


