---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.inline.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) _INTERRUPTIBLE マクロの中身を展開するだけ.
      ---------------------------------------- -}

	// Don't attend to UseVMInterruptibleIO. Always allow interruption.
	// Also assumes that it is called from the _thread_blocked state.
	// Used by os_sleep().
	
	#define INTERRUPTIBLE_NORESTART_VM_ALWAYS(_cmd, _result, _thread, _clear) \
	  _INTERRUPTIBLE(os::Solaris::setup_interruptible_already_blocked(_thread), _result = _cmd, _result, _thread, _clear, , , true )
	
```


