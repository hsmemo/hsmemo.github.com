---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp
### 説明(description)

```
// name - pre-checked for NULL
// monitor_ptr - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::CreateRawMonitor(const char* name, jrawMonitorID* monitor_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい JvmtiRawMonitor オブジェクトを確保し, 
      そのポインタを monitor_ptr 引数で指定された箇所にセットするだけ.
  
      (なお, 確保に失敗したら JVMTI_ERROR_OUT_OF_MEMORY エラーをリターン)
      ---------------------------------------- -}

	  JvmtiRawMonitor* rmonitor = new JvmtiRawMonitor(name);
	  NULL_CHECK(rmonitor, JVMTI_ERROR_OUT_OF_MEMORY);
	
	  *monitor_ptr = (jrawMonitorID)rmonitor;
	
	  return JVMTI_ERROR_NONE;
	} /* end CreateRawMonitor */
	
```


