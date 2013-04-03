---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp
### 説明(description)
この書き換え処理は VM Operation 中で実行するが, 
ネイティブメソッドを実行中のスレッドは停止しないので
念のため関数ポインタはアトミックに書き換えておく.

```
// For jvmti use to modify jni function table.
// Java threads in native contiues to run until it is transitioned
// to VM at safepoint. Before the transition or before it is blocked
// for safepoint it may access jni function table. VM could crash if
// any java thread access the jni function table in the middle of memcpy.
// To avoid this each function pointers are copied automically.
```

### 名前(function name)
```
void copy_jni_function_table(const struct JNINativeInterface_ *new_jni_NativeInterface) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint(), "must be at safepoint");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  intptr_t *a = (intptr_t *) jni_functions();
	  intptr_t *b = (intptr_t *) new_jni_NativeInterface;

  {- -------------------------------------------
  (1) new_jni_NativeInterface 引数で指定された内容で
      JNI Function table を上書きする.
      (なお, 各関数ポインタは Atomic::store_ptr() でアトミックに書き込む)
      ---------------------------------------- -}

	  for (uint i=0; i <  sizeof(struct JNINativeInterface_)/sizeof(void *); i++) {
	    Atomic::store_ptr(*b++, a++);
	  }
	}
	
```


