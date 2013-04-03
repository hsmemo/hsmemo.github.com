---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvThreadState.cpp
### 説明(description)
(コメントでは二重送信を防止するための関数と書かれているが, どんなケースで二重送信になる?? #TODO
 bytecode 命令が融合されている時(?)やシングルステップと併用されている時のようだが...??)

```
// Given that a new (potential) event has come in,
// maintain the current JVMTI location on a per-thread per-env basis
// and use it to filter out duplicate events:
// - instruction rewrites
// - breakpoint followed by single step
// - single step at a breakpoint
```

### 名前(function name)
```
void JvmtiEnvThreadState::compare_and_set_current_location(methodOop new_method,
                                                           address new_location, jvmtiEvent event) {
```

### 本体部(body)
```
	
	  int new_bci = new_location - new_method->code_base();
	
	  // The method is identified and stored as a jmethodID which is safe in this
	  // case because the class cannot be unloaded while a method is executing.
	  jmethodID new_method_id = new_method->jmethod_id();
	
	  // the last breakpoint or single step was at this same location
	  if (_current_bci == new_bci && _current_method_id == new_method_id) {
	    switch (event) {
	    case JVMTI_EVENT_BREAKPOINT:
	      // Repeat breakpoint is complicated. If we previously posted a breakpoint
	      // event at this location and if we also single stepped at this location
	      // then we skip the duplicate breakpoint.
	      _breakpoint_posted = _breakpoint_posted && _single_stepping_posted;
	      break;
	    case JVMTI_EVENT_SINGLE_STEP:
	      // Repeat single step is easy: just don't post it again.
	      // If step is pending for popframe then it may not be
	      // a repeat step. The new_bci and method_id is same as current_bci
	      // and current method_id after pop and step for recursive calls.
	      // This has been handled by clearing the location
	      _single_stepping_posted = true;
	      break;
	    default:
	      assert(false, "invalid event value passed");
	      break;
	    }
	    return;
	  }
	
	  set_current_location(new_method_id, new_bci);
	  _breakpoint_posted = false;
	  _single_stepping_posted = false;
	}
	
```


