---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/native/java/lang/ClassLoader.c

### 名前(function name)
```
static jboolean initIDs(JNIEnv *env)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) まだ初期化が終わっていなければ (handleID が 0 のままあれば), 
      以下の 2つの static 変数を初期化する.
        * handleID
        * jniVersionID
  
      もし初期化にエラーが起きたら JNI_FALSE をリターン.
      (そうでなければ, JNI_TRUE をリターン)
      ---------------------------------------- -}

	    if (handleID == 0) {
	        jclass this =
	            (*env)->FindClass(env, "java/lang/ClassLoader$NativeLibrary");
	        if (this == 0)
	            return JNI_FALSE;
	        handleID = (*env)->GetFieldID(env, this, "handle", "J");
	        if (handleID == 0)
	            return JNI_FALSE;
	        jniVersionID = (*env)->GetFieldID(env, this, "jniVersion", "I");
	        if (jniVersionID == 0)
	            return JNI_FALSE;
	    }
	    return JNI_TRUE;
	}
	
```


