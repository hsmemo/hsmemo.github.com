---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp
### 説明(description)


```
// -----------------------------------------------------------------------------
// A macro is used below because there may already be a pending
// exception which should not abort the execution of the routines
// which use this (which is why we don't put this into check_slow and
// call it with a CHECK argument).
```

### 名前(function name)
```
#define CHECK_OWNER()                                                             \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ロックを持っていなければ IllegalMonitorStateException を投げる.
  
      (また, もしカレントスレッドがロックを持っているにも関わらず
       ObjectMonitor 内の _owner フィールドがカレントスレッドを指していない場合, 
      _owner をカレントスレッドに変更している.
  
       これは, stack-locked 状態のロックが inflate された直後は, 
       _owner に正しい値が入っていない為に必要な措置.
       See: ObjectSynchronizer::inflate())
      ---------------------------------------- -}

	  do {                                                                            \
	    if (THREAD != _owner) {                                                       \
	      if (THREAD->is_lock_owned((address) _owner)) {                              \
	        _owner = THREAD ;  /* Convert from basiclock addr to Thread addr */       \
	        _recursions = 0;                                                          \
	        OwnerIsThread = 1 ;                                                       \
	      } else {                                                                    \
	        TEVENT (Throw IMSX) ;                                                     \
	        THROW(vmSymbols::java_lang_IllegalMonitorStateException());               \
	      }                                                                           \
	    }                                                                             \
	  } while (false)
	
```


