---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) false をリターンするだけ.
      ---------------------------------------- -}

	  // It is possible to get the receiver out of a non-static native wrapper
	  // frame.  Use VM_GetReceiver to do this.
	  virtual bool getting_receiver() const { return false; }
	
```


