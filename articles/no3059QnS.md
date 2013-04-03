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
JNI_ENTRY(jclass, jni_FindClass(JNIEnv *env, const char *name))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("FindClass");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, FindClass__entry, env, name);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jclass result = NULL;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(FindClass, jclass, (const jclass&)result);
	
  {- -------------------------------------------
  (1) (first_time_FindClass は, 初期値が true の static 変数.
       ここで false にしてしまうため, jni_FindClass が最初に呼ばれた時のみ true になっている.
       この値は後で使用する.)
      ---------------------------------------- -}

	  // Remember if we are the first invocation of jni_FindClass
	  bool first_time = first_time_FindClass;
	  first_time_FindClass = false;
	
  {- -------------------------------------------
  (1) クラス名が NULL だったり, あるいは
      クラス名が長すぎる場合(Symbol::max_length() を超えている場合)は, NoClassDefFoundError.
      ---------------------------------------- -}

	  // Sanity check the name:  it cannot be null or larger than the maximum size
	  // name we can fit in the constant pool.
	  if (name == NULL || (int)strlen(name) > Symbol::max_length()) {
	    THROW_MSG_0(vmSymbols::java_lang_NoClassDefFoundError(), name);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (loader は, クラスの探索時に使用するクラスローダ.
       protection_domain は, クラスの探索時に使用する protection domain)
      ---------------------------------------- -}

	  //%note jni_3
	  Handle loader;
	  Handle protection_domain;

  {- -------------------------------------------
  (1) クラスの探索時に使用するクラスローダ(loader)と protection_domain を適切に設定しておく.
      #TODO
      ---------------------------------------- -}

	  // Find calling class
	  instanceKlassHandle k (THREAD, thread->security_get_caller_class(0));
	  if (k.not_null()) {
	    loader = Handle(THREAD, k->class_loader());
	    // Special handling to make sure JNI_OnLoad and JNI_OnUnload are executed
	    // in the correct class context.
	    if (loader.is_null() &&
	        k->name() == vmSymbols::java_lang_ClassLoader_NativeLibrary()) {
	      JavaValue result(T_OBJECT);
	      JavaCalls::call_static(&result, k,
	                                      vmSymbols::getFromClass_name(),
	                                      vmSymbols::void_class_signature(),
	                                      thread);
	      if (HAS_PENDING_EXCEPTION) {
	        Handle ex(thread, thread->pending_exception());
	        CLEAR_PENDING_EXCEPTION;
	        THROW_HANDLE_0(ex);
	      }
	      oop mirror = (oop) result.get_jobject();
	      loader = Handle(THREAD,
	        instanceKlass::cast(java_lang_Class::as_klassOop(mirror))->class_loader());
	      protection_domain = Handle(THREAD,
	        instanceKlass::cast(java_lang_Class::as_klassOop(mirror))->protection_domain());
	    }
	  } else {
	    // We call ClassLoader.getSystemClassLoader to obtain the system class loader.
	    loader = Handle(THREAD, SystemDictionary::java_system_loader());
	  }
	
  {- -------------------------------------------
  (1) find_class_from_class_loader() を呼んで, 指定されたクラスを探し出す.
      ---------------------------------------- -}

	  TempNewSymbol sym = SymbolTable::new_symbol(name, CHECK_NULL);
	  result = find_class_from_class_loader(env, sym, true, loader,
	                                        protection_domain, true, thread);
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceClassResolution && result != NULL) {
	    trace_class_resolution(java_lang_Class::as_klassOop(JNIHandles::resolve_non_null(result)));
	  }
	
  {- -------------------------------------------
  (1) もし今回が jni_FindClass の初回の呼び出しであれば, 
      CompilationPolicy::completed_vm_startup() を呼び出して JIT コンパイルが有効な状態にしておく.
      ---------------------------------------- -}

	  // If we were the first invocation of jni_FindClass, we enable compilation again
	  // rather than just allowing invocation counter to overflow and decay.
	  // Controlled by flag DelayCompilationDuringStartup.
	  if (first_time && !CompileTheWorld)
	    CompilationPolicy::completed_vm_startup();
	
  {- -------------------------------------------
  (1) 結果をリターン.
      ---------------------------------------- -}

	  return result;
	JNI_END
	
```


