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
JNIEXPORT jclass JNICALL
Java_java_lang_ClassLoader_findLoadedClass0(JNIEnv *env, jobject loader,
                                           jstring name)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JVM_FindLoadedClass() を呼んで既にロード済みかどうかをチェックし, 結果をリターン.
  
      (ただし, name 引数が NULL の場合は何もせずにリターンするだけ)
      ---------------------------------------- -}

	    if (name == NULL) {
	        return 0;
	    } else {
	        return JVM_FindLoadedClass(env, loader, name);
	    }
	}
	
```


