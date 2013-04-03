---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp

### 名前(function name)
```
os::YieldResult os::NakedYield() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) os::Kernel32Dll::SwitchToThreadAvailable() の返り値に応じて, 
      以下のどちらかのシステムコールを呼び出す.
      (true なら SwitchToThread(), false なら Sleep())
      ---------------------------------------- -}

	  // Use either SwitchToThread() or Sleep(0)
	  // Consider passing back the return value from SwitchToThread().
	  if (os::Kernel32Dll::SwitchToThreadAvailable()) {
	    return SwitchToThread() ? os::YIELD_SWITCHED : os::YIELD_NONEREADY ;
	  } else {
	    Sleep(0);
	  }
	  return os::YIELD_UNKNOWN ;
	}
	
```


