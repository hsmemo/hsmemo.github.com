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
// common code for JVM_DefineClass() and JVM_DefineClassWithSource()
// and JVM_DefineClassWithSourceCond()
```

### 名前(function name)
```
static jclass jvm_define_class_common(JNIEnv *env, const char *name,
                                      jobject loader, const jbyte *buf,
                                      jsize len, jobject pd, const char *source,
                                      jboolean verify, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (source == NULL)  source = "__JVM_DefineClass__";
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(THREAD->is_Java_thread(), "must be a JavaThread");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread* jt = (JavaThread*) THREAD;
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) (See: PerfClassTraceTime)
      ---------------------------------------- -}

	  PerfClassTraceTime vmtimer(ClassLoader::perf_define_appclass_time(),
	                             ClassLoader::perf_define_appclass_selftime(),
	                             ClassLoader::perf_define_appclasses(),
	                             jt->get_thread_stat()->perf_recursion_counts_addr(),
	                             jt->get_thread_stat()->perf_timers_addr(),
	                             PerfClassTraceTime::DEFINE_CLASS);
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) ("sun.cls.appClassBytes")
      ---------------------------------------- -}

	  if (UsePerfData) {
	    ClassLoader::perf_app_classfile_bytes_read()->inc(len);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (なお, name 引数の文字列が長すぎる場合は NoClassDefFoundError)
      ---------------------------------------- -}

	  // Since exceptions can be thrown, class initialization can take place
	  // if name is NULL no check for class name in .class stream has to be made.
	  TempNewSymbol class_name = NULL;
	  if (name != NULL) {
	    const int str_len = (int)strlen(name);
	    if (str_len > Symbol::max_length()) {
	      // It's impossible to create this class;  the name cannot fit
	      // into the constant pool.
	      THROW_MSG_0(vmSymbols::java_lang_NoClassDefFoundError(), name);
	    }
	    class_name = SymbolTable::new_symbol(name, str_len, CHECK_NULL);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);
	  ClassFileStream st((u1*) buf, len, (char *)source);
	  Handle class_loader (THREAD, JNIHandles::resolve(loader));

  {- -------------------------------------------
  (1) (プロファイル情報の記録) ("sun.cls.jvmDefineClassNoLockCalls")
      (See: ClassLoader::sync_JVMDefineClassLockFreeCounter())
      ---------------------------------------- -}

	  if (UsePerfData) {
	    is_lock_held_by_thread(class_loader,
	                           ClassLoader::sync_JVMDefineClassLockFreeCounter(),
	                           THREAD);
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Handle protection_domain (THREAD, JNIHandles::resolve(pd));

  {- -------------------------------------------
  (1) SystemDictionary::resolve_from_stream() を呼んで, クラスのロード処理を行う.
      ---------------------------------------- -}

	  klassOop k = SystemDictionary::resolve_from_stream(class_name, class_loader,
	                                                     protection_domain, &st,
	                                                     verify != 0,
	                                                     CHECK_NULL);
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceClassResolution && k != NULL) {
	    trace_class_resolution(k);
	  }
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return (jclass) JNIHandles::make_local(env, Klass::cast(k)->java_mirror());
	}
	
```


