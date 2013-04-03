---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp

### 名前(function name)
```
int JvmtiBreakpoints::set(JvmtiBreakpoint& bp) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし既にブレークポイントになっていれば(= 既に _bps 内に記録されていれば), ここでリターン(JVMTI_ERROR_DUPLICATE).
      ---------------------------------------- -}

	  if ( _bps.find(bp) != -1) {
	     return JVMTI_ERROR_DUPLICATE;
	  }

  {- -------------------------------------------
  (1) VM_ChangeBreakpoints::doit() で, 指定された箇所をブレークポイントにする.
      (ブレークポイントを設定するので, 引数には VM_ChangeBreakpoints::SET_BREAKPOINT を指定)
      ---------------------------------------- -}

	  VM_ChangeBreakpoints set_breakpoint(this,VM_ChangeBreakpoints::SET_BREAKPOINT, &bp);
	  VMThread::execute(&set_breakpoint);

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	}
	
```


