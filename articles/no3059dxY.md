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
JNI_ENTRY(jclass, jni_DefineClass(JNIEnv *env, const char *name, jobject loaderRef,
                                  const jbyte *buf, jsize bufLen))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("DefineClass");
	
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE5(hotspot_jni, DefineClass__entry,
	    env, name, loaderRef, buf, bufLen);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jclass cls = NULL;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(DefineClass, jclass, (const jclass&)cls);
	
  {- -------------------------------------------
  (1) もしクラス名が長すぎる場合 (Symbol::max_length() を超えている場合) は, NoClassDefFoundError.
      ---------------------------------------- -}

	  // Since exceptions can be thrown, class initialization can take place
	  // if name is NULL no check for class name in .class stream has to be made.
	  if (name != NULL) {
	    const int str_len = (int)strlen(name);
	    if (str_len > Symbol::max_length()) {
	      // It's impossible to create this class;  the name cannot fit
	      // into the constant pool.
	      THROW_MSG_0(vmSymbols::java_lang_NoClassDefFoundError(), name);
	    }
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  TempNewSymbol class_name = SymbolTable::new_symbol(name, THREAD);
	
	  ResourceMark rm(THREAD);
	  ClassFileStream st((u1*) buf, bufLen, NULL);
	  Handle class_loader (THREAD, JNIHandles::resolve(loaderRef));
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) ("sun.cls.jniDefineClassNoLockCalls") (See: UsePerfData)
      ---------------------------------------- -}

	  if (UsePerfData && !class_loader.is_null()) {
	    // check whether the current caller thread holds the lock or not.
	    // If not, increment the corresponding counter
	    if (ObjectSynchronizer::
	        query_lock_ownership((JavaThread*)THREAD, class_loader) !=
	        ObjectSynchronizer::owner_self) {
	      ClassLoader::sync_JNIDefineClassLockFreeCounter()->inc();
	    }
	  }

  {- -------------------------------------------
  (1) SystemDictionary::resolve_from_stream() を呼び出して, クラスの生成を行う.
      ---------------------------------------- -}

	  klassOop k = SystemDictionary::resolve_from_stream(class_name, class_loader,
	                                                     Handle(), &st, true,
	                                                     CHECK_NULL);
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceClassResolution && k != NULL) {
	    trace_class_resolution(k);
	  }
	
  {- -------------------------------------------
  (1) 結果を JNI Handle 化してリターン.
      ---------------------------------------- -}

	  cls = (jclass)JNIHandles::make_local(
	    env, Klass::cast(k)->java_mirror());
	  return cls;
	JNI_END
	
```


