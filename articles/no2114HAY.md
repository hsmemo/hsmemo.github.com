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
  void do_monitor(ObjectMonitor* mid) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下の2つの条件がどちらも成り立てば, それはネイティブメソッド内で取得したモニターだと判断して, 
      ThreadStackTrace::add_jni_locked_monitor() を呼び出す.
      * owner が自分である ObjectMonitor がある
      * しかし (Java のメソッド内で取得したロックに関しては全て記録済みの) ThreadStackTrace 内に
        その ObjectMonitor の情報が無い (= ThreadStackTrace::is_owned_monitor_on_stack() が false)
      ---------------------------------------- -}

	    if (mid->owner() == _thread) {
	      oop object = (oop) mid->object();
	      if (!_stack_trace->is_owned_monitor_on_stack(object)) {
	        _stack_trace->add_jni_locked_monitor(object);
	      }
	    }
	  }
	
```


