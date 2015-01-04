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
JVM_ENTRY(jclass, JVM_FindLoadedClass(JNIEnv *env, jobject loader, jstring name))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper)
      ---------------------------------------- -}

	  JVMWrapper("JVM_FindLoadedClass");

  {- -------------------------------------------
  (1) (変数宣言など)
      (klass_name 局所変数は, name 引数で指定されたクラス名に対応する TempNewSymbol)
  
      (なお, name 引数の文字列がクラス名としては長すぎる場合 (Symbol::max_length() を超えている場合) は
      ここでリターン(返値は NULL))
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);
	
	  Handle h_name (THREAD, JNIHandles::resolve_non_null(name));
	  Handle string = java_lang_String::internalize_classname(h_name, CHECK_NULL);
	
	  const char* str   = java_lang_String::as_utf8_string(string());
	  // Sanity check, don't expect null
	  if (str == NULL) return NULL;
	
	  const int str_len = (int)strlen(str);
	  if (str_len > Symbol::max_length()) {
	    // It's impossible to create this class;  the name cannot fit
	    // into the constant pool.
	    return NULL;
	  }
	  TempNewSymbol klass_name = SymbolTable::new_symbol(str, str_len, CHECK_NULL);
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) ("sun.cls.jvmFindLoadedClassNoLockCalls")
      (See: ClassLoader::sync_JVMFindLoadedClassLockFreeCounter())
      ---------------------------------------- -}

	  // Security Note:
	  //   The Java level wrapper will perform the necessary security check allowing
	  //   us to pass the NULL as the initiating class loader.
	  Handle h_loader(THREAD, JNIHandles::resolve(loader));
	  if (UsePerfData) {
	    is_lock_held_by_thread(h_loader,
	                           ClassLoader::sync_JVMFindLoadedClassLockFreeCounter(),
	                           THREAD);
	  }
	
  {- -------------------------------------------
  (1) SystemDictionary::find_instance_or_array_klass() を呼んで
      既にロード済みかどうかをチェックし, 結果をリターン.
      (ロード済みであれば, 対象クラスの mirror オブジェクトを JNI Handle 化してリターン.
      ロード済みでなければ NULL をリターン)
      ---------------------------------------- -}

	  klassOop k = SystemDictionary::find_instance_or_array_klass(klass_name,
	                                                              h_loader,
	                                                              Handle(),
	                                                              CHECK_NULL);
	
	  return (k == NULL) ? NULL :
	            (jclass) JNIHandles::make_local(env, Klass::cast(k)->java_mirror());
	JVM_END
	
```


