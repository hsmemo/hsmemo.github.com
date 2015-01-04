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
JNIEXPORT void JNICALL
Java_java_lang_ClassLoader_resolveClass0(JNIEnv *env, jobject this,
                                         jclass cls)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) cls 引数が不正な場合は NullPointerException.
      ---------------------------------------- -}

	    if (cls == NULL) {
	        JNU_ThrowNullPointerException(env, 0);
	        return;
	    }
	
  {- -------------------------------------------
  (1) JVM_ResolveClass() を呼び出す.
      ---------------------------------------- -}

	    JVM_ResolveClass(env, cls);
	}
	
```


