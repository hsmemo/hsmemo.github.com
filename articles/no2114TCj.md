---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.hpp

### 名前(function name)
```
  jlong cooked_allocated_bytes() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Thread::_allocated_bytes フィールドの値をリターンする.
      (ただし, UseTLAB 時には, さらに現在の TLAB の使用量も加えた値をリターンする)
      ---------------------------------------- -}

	    jlong allocated_bytes = OrderAccess::load_acquire(&_allocated_bytes);
	    if (UseTLAB) {
	      size_t used_bytes = tlab().used_bytes();
	      if ((ssize_t)used_bytes > 0) {
	        // More-or-less valid tlab.  The load_acquire above should ensure
	        // that the result of the add is <= the instantaneous value
	        return allocated_bytes + used_bytes;
	      }
	    }
	    return allocated_bytes;
	  }
	
```


