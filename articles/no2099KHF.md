---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp
### 説明(description)

```
// Returns a class loaded by the bootstrap class loader; or null
// if not found.  ClassNotFoundException is not thrown.
//
// Rationale behind JVM_FindClassFromBootLoader
// a> JVM_FindClassFromClassLoader was never exported in the export tables.
// b> because of (a) java.dll has a direct dependecy on the  unexported
//    private symbol "_JVM_FindClassFromClassLoader@20".
// c> the launcher cannot use the private symbol as it dynamically opens
//    the entry point, so if something changes, the launcher will fail
//    unexpectedly at runtime, it is safest for the launcher to dlopen a
//    stable exported interface.
// d> re-exporting JVM_FindClassFromClassLoader as public, will cause its
//    signature to change from _JVM_FindClassFromClassLoader@20 to
//    JVM_FindClassFromClassLoader and will not be backward compatible
//    with older JDKs.
// Thus a public/stable exported entry point is the right solution,
// public here means public in linker semantics, and is exported only
// to the JDK, and is not intended to be a public API.

```

### 名前(function name)
```
JVM_ENTRY(jclass, JVM_FindClassFromBootLoader(JNIEnv* env,
                                              const char* name))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper2)
      ---------------------------------------- -}

	  JVMWrapper2("JVM_FindClassFromBootLoader %s", name);
	
  {- -------------------------------------------
  (1) name 引数が不正であれば, ここでリターン.
      ---------------------------------------- -}

	  // Java libraries should ensure that name is never null...
	  if (name == NULL || (int)strlen(name) > Symbol::max_length()) {
	    // It's impossible to create this class;  the name cannot fit
	    // into the constant pool.
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  TempNewSymbol h_name = SymbolTable::new_symbol(name, CHECK_NULL);

  {- -------------------------------------------
  (1) SystemDictionary::resolve_or_null() を呼んで, name 引数で指定されたクラスをロードする.
      (失敗したら, ここでリターン)
      ---------------------------------------- -}

	  klassOop k = SystemDictionary::resolve_or_null(h_name, CHECK_NULL);
	  if (k == NULL) {
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceClassResolution) {
	    trace_class_resolution(k);
	  }

  {- -------------------------------------------
  (1) ロードしたクラスの mirror オブジェクトを JNI Handle 化し, リターン.
      ---------------------------------------- -}

	  return (jclass) JNIHandles::make_local(env, Klass::cast(k)->java_mirror());
	JVM_END
	
```


