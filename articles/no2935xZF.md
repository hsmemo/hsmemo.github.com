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
// function_table - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::GetJNIFunctionTable(jniNativeInterface** function_table) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 結果を格納するためのメモリを確保する.
      (確保が失敗したら, ここでリターン(JVMTI_ERROR_OUT_OF_MEMORY))
      ---------------------------------------- -}

	  *function_table=(jniNativeInterface*)jvmtiMalloc(sizeof(jniNativeInterface));
	  if (*function_table == NULL)
	    return JVMTI_ERROR_OUT_OF_MEMORY;

  {- -------------------------------------------
  (1) memcpy() で, JNI Function table の内容を
      function_table 引数で指定された箇所にコピーする.
      ---------------------------------------- -}

	  memcpy(*function_table,(JavaThread::current())->get_jni_functions(),sizeof(jniNativeInterface));

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	} /* end GetJNIFunctionTable */
	
```


