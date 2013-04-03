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
// method_oop - pre-checked for validity, but may be NULL meaning obsolete method
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::ClearBreakpoint(methodOop method_oop, jlocation location) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) method_oop 引数が NULL の場合は, ここでリターン(JVMTI_ERROR_INVALID_METHODID).
      ---------------------------------------- -}

	  NULL_CHECK(method_oop, JVMTI_ERROR_INVALID_METHODID);
	
  {- -------------------------------------------
  (1) location 引数が負値だったりコードサイズを越えていたら, ここでリターン(JVMTI_ERROR_INVALID_LOCATION).
      ---------------------------------------- -}

	  if (location < 0) {   // simple invalid location check first
	    return JVMTI_ERROR_INVALID_LOCATION;
	  }
	
	  // verify that the breakpoint is not past the end of the method
	  if (location >= (jlocation) method_oop->code_size()) {
	    return JVMTI_ERROR_INVALID_LOCATION;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
  
      (なお, JvmtiBreakpoints オブジェクトが作られていなかった場合は
       この JvmtiCurrentBreakpoints::get_jvmti_breakpoints() 呼び出し時に遅延生成される)
      ---------------------------------------- -}

	  JvmtiBreakpoint bp(method_oop, location);
	
	  JvmtiBreakpoints& jvmti_breakpoints = JvmtiCurrentBreakpoints::get_jvmti_breakpoints();

  {- -------------------------------------------
  (1) JvmtiBreakpoints::clear() を呼んで, 引数で指定された地点をブレークポイントから元に戻す.
      (なお, 指定箇所がブレークポイント命令でなければ, ここでリターン(JVMTI_ERROR_NOT_FOUND))
      ---------------------------------------- -}

	  if (jvmti_breakpoints.clear(bp) == JVMTI_ERROR_NOT_FOUND)
	    return JVMTI_ERROR_NOT_FOUND;
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceJVMTICalls) {
	    jvmti_breakpoints.print();
	  }
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	} /* end ClearBreakpoint */
	
```


