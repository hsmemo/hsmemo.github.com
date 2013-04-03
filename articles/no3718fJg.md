---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp
### 説明(description)

```
// Must be JNI_ENTRY (with HandleMark)
```

### 名前(function name)
```
JNI_ENTRY_NO_PRESERVE(void, jni_DeleteGlobalRef(JNIEnv *env, jobject ref))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("DeleteGlobalRef");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, DeleteGlobalRef__entry, env, ref);

  {- -------------------------------------------
  (1) JNIHandles::destroy_global() で
      JNIHandles::_global_handles 内の該当する箇所に JNIHandles::_deleted_handle の値を書き込む.
      ---------------------------------------- -}

	  JNIHandles::destroy_global(ref);

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE(hotspot_jni, DeleteGlobalRef__return);
	JNI_END
	
```


