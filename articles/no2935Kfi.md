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
int JvmtiBreakpoints::clear(JvmtiBreakpoint& bp) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし指定箇所がブレークポイント命令でなければ(= _bps 内に記録されていなければ), 
      ここでリターン(JVMTI_ERROR_NOT_FOUND).
      ---------------------------------------- -}

	  if ( _bps.find(bp) == -1) {
	     return JVMTI_ERROR_NOT_FOUND;
	  }
	
  {- -------------------------------------------
  (1) VM_ChangeBreakpoints::doit() で, 指定された箇所をブレークポイントから元に戻す.
      (元に戻すので, 引数には VM_ChangeBreakpoints::CLEAR_BREAKPOINT を指定)
      ---------------------------------------- -}

	  VM_ChangeBreakpoints clear_breakpoint(this,VM_ChangeBreakpoints::CLEAR_BREAKPOINT, &bp);
	  VMThread::execute(&clear_breakpoint);
	  return JVMTI_ERROR_NONE;
	}
	
```


