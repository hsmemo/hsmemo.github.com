---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp

### 名前(function name)
```
void ObjectWaiter::wait_reenter_end(ObjectMonitor *mon) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JavaThreadBlockedOnMonitorEnterState::wait_reenter_end() を呼び出すだけ
      (See: JavaThreadBlockedOnMonitorEnterState) (See: [here](no21146np.html) for details)
      ---------------------------------------- -}

	  JavaThread *jt = (JavaThread *)this->_thread;
	  JavaThreadBlockedOnMonitorEnterState::wait_reenter_end(jt, _active);
	}
	
```


