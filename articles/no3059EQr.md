---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp

### 名前(function name)
```
jclass find_class_from_class_loader(JNIEnv* env, Symbol* name, jboolean init, Handle loader, Handle protection_domain, jboolean throwError, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Security Note:
	  //   The Java level wrapper will perform the necessary security check allowing
	  //   us to pass the NULL as the initiating class loader.

  {- -------------------------------------------
  (1) SystemDictionary::resolve_or_fail() を呼び出して, 指定されたクラスを取得する.
      ---------------------------------------- -}

	  klassOop klass = SystemDictionary::resolve_or_fail(name, loader, protection_domain, throwError != 0, CHECK_NULL);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  KlassHandle klass_handle(THREAD, klass);

  {- -------------------------------------------
  (1) 引数で初期化も行うように指定されていれば, かつ...#TODO
      Klass::initialize() で初期化しておく.
      ---------------------------------------- -}

	  // Check if we should initialize the class
	  if (init && klass_handle->oop_is_instance()) {
	    klass_handle->initialize(CHECK_NULL);
	  }

  {- -------------------------------------------
  (1) 結果を JNI Handle 化してリターン.
      ---------------------------------------- -}

	  return (jclass) JNIHandles::make_local(env, klass_handle->java_mirror());
	}
	
```


