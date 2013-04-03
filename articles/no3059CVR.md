---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp

### 名前(function name)
```
JNI_LEAF(jint, jni_EnsureLocalCapacity(JNIEnv *env, jint capacity))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("EnsureLocalCapacity");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, EnsureLocalCapacity__entry, env, capacity);

  {- -------------------------------------------
  (1) 引数で指定された容量が 0 以上 MAX_REASONABLE_LOCAL_CAPACITY 以下であれば, 返値は JNI_OK とする.
      そうでなければ, 返値は JNI_ERR とする.
      ---------------------------------------- -}

	  jint ret;
	  if (capacity >= 0 && capacity <= MAX_REASONABLE_LOCAL_CAPACITY) {
	    ret = JNI_OK;
	  } else {
	    ret = JNI_ERR;
	  }

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, EnsureLocalCapacity__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


