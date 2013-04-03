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
JNI_ENTRY(jint, jni_PushLocalFrame(JNIEnv *env, jint capacity))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("PushLocalFrame");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, PushLocalFrame__entry, env, capacity);
	  //%note jni_11

  {- -------------------------------------------
  (1) もし引数が不正であれば, ここでリターン.
      ---------------------------------------- -}

	  if (capacity < 0 && capacity > MAX_REASONABLE_LOCAL_CAPACITY) {

    {- -------------------------------------------
  (1.1) (DTrace のフック点)
        ---------------------------------------- -}

	    DTRACE_PROBE1(hotspot_jni, PushLocalFrame__return, JNI_ERR);
	    return JNI_ERR;
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JNIHandleBlock* old_handles = thread->active_handles();

  {- -------------------------------------------
  (1) JNIHandleBlock::allocate_block() で新しい JNIHandleBlock を確保し, 
      JNIHandleBlock::set_pop_frame_link() で
      新しい JNIHandleBlock オブジェクトの _pop_frame_link フィールドに古いブロックをつなぐ.
      ---------------------------------------- -}

	  JNIHandleBlock* new_handles = JNIHandleBlock::allocate_block(thread);
	  assert(new_handles != NULL, "should not be NULL");
	  new_handles->set_pop_frame_link(old_handles);

  {- -------------------------------------------
  (1) 新しい JNIHandleBlock オブジェクトをこのスレッドの active_handles に設定.
      ---------------------------------------- -}

	  thread->set_active_handles(new_handles);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jint ret = JNI_OK;

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, PushLocalFrame__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン.
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


