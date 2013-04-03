---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp
### 説明(description)

```
// Fills names with VM internal thread names and times with the corresponding
// CPU times.  If names or times is NULL, a NullPointerException is thrown.
// If the element type of names is not String, an IllegalArgumentException is
// thrown.
// If an array is not large enough to hold all the entries, only the entries
// that fit will be returned.  Return value is the number of VM internal
// threads entries.
```

### 名前(function name)
```
JVM_ENTRY(jint, jmm_GetInternalThreadTimes(JNIEnv *env,
                                           jobjectArray names,
                                           jlongArray times))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし, 配列であるべき引数が実は NULL だった場合, NullPointerException.
      ---------------------------------------- -}

	  if (names == NULL || times == NULL) {
	     THROW_(vmSymbols::java_lang_NullPointerException(), 0);
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  objArrayOop na = objArrayOop(JNIHandles::resolve_non_null(names));
	  objArrayHandle names_ah(THREAD, na);
	
  {- -------------------------------------------
  (1) 引数(names)の型をチェックしておく.
      もし String 配列でなければ IllegalArgumentException.
      ---------------------------------------- -}

	  // Make sure we have a String array
	  klassOop element_klass = objArrayKlass::cast(names_ah->klass())->element_klass();
	  if (element_klass != SystemDictionary::String_klass()) {
	    THROW_MSG_(vmSymbols::java_lang_IllegalArgumentException(),
	               "Array element type is not String class", 0);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  typeArrayOop ta = typeArrayOop(JNIHandles::resolve_non_null(times));
	  typeArrayHandle times_ah(THREAD, ta);
	
  {- -------------------------------------------
  (1) ThreadTimesClosure によって, 
      引数で渡されていた配列(names, times)の中に
      各スレッドのスレッド名と CPU 使用時間を格納し, 
      その結果をリターン.
      ---------------------------------------- -}

	  ThreadTimesClosure ttc(names_ah(), times_ah());
	  {
	    MutexLockerEx ml(Threads_lock);
	    Threads::threads_do(&ttc);
	  }
	
	  return ttc.count();
	JVM_END
	
```


