---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/safepoint.cpp

### 名前(function name)
```
bool SafepointSynchronize::safepoint_safe(JavaThread *thread, JavaThreadState state) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下の 2つのケースでは true をリターンする (それ以外のケースでは false をリターンする)
      
      * JavaThreadState が _thread_in_native で, かつ, 
        Java レベルのメソッドのスタックフレームを持っていないか (= !thread->has_last_Java_frame()), 
        持っていても walkable である場合 (= thread->frame_anchor()->walkable()):
  
      * JavaThreadState が _thread_blocked の場合:
      ---------------------------------------- -}

	  switch(state) {
	  case _thread_in_native:
	    // native threads are safe if they have no java stack or have walkable stack
	    return !thread->has_last_Java_frame() || thread->frame_anchor()->walkable();
	
	   // blocked threads should have already have walkable stack
	  case _thread_blocked:
	    assert(!thread->has_last_Java_frame() || thread->frame_anchor()->walkable(), "blocked and not walkable");
	    return true;
	
	  default:
	    return false;
	  }
	}
	
```


