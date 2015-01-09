---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/compiler/compileBroker.cpp

### 名前(function name)
```
nmethod* CompileBroker::compile_method(methodHandle method, int osr_bci,
                                       int comp_level,
                                       methodHandle hot_method, int hot_count,
                                       const char* comment, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // make sure arguments make sense
	  assert(method->method_holder()->klass_part()->oop_is_instance(), "not an instance method");
	  assert(osr_bci == InvocationEntryBci || (0 <= osr_bci && osr_bci < method->code_size()), "bci out of range");
	  assert(!method->is_abstract() && (osr_bci == InvocationEntryBci || !method->is_native()), "cannot compile abstract/native methods");
	  assert(!instanceKlass::cast(method->method_holder())->is_not_initialized(), "method holder must be initialized");
	
  {- -------------------------------------------
  (1) 引数を調整
      ---------------------------------------- -}

	  if (!TieredCompilation) {
	    comp_level = CompLevel_highest_tier;
	  }
	
  {- -------------------------------------------
  (1) もし該当するレベルの JIT コンパイラが存在しなかったり,
      対象が JIT コンパイルが禁止されているものだった場合は, 
      ここでリターン.
      ---------------------------------------- -}

	  // return quickly if possible
	
	  // lock, make sure that the compilation
	  // isn't prohibited in a straightforward way.
	
	  if (compiler(comp_level) == NULL || compilation_is_prohibited(method, osr_bci, comp_level)) {
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) もし既に JIT コンパイルが終わっていた場合は, その結果をリターン.
      また, コンパイルが禁止されているものの場合は, NULL をリターン.
      ---------------------------------------- -}

	  if (osr_bci == InvocationEntryBci) {
	    // standard compilation
	    nmethod* method_code = method->code();
	    if (method_code != NULL) {
	      if (compilation_is_complete(method, osr_bci, comp_level)) {
	        return method_code;
	      }
	    }
	    if (method->is_not_compilable(comp_level)) return NULL;
	
	    if (UseCodeCacheFlushing) {
	      nmethod* saved = CodeCache::find_and_remove_saved_code(method());
	      if (saved != NULL) {
	        method->set_code(method, saved);
	        return saved;
	      }
	    }
	
	  } else {
	    // osr compilation
	#ifndef TIERED
	    // seems like an assert of dubious value
	    assert(comp_level == CompLevel_highest_tier,
	           "all OSR compiles are assumed to be at a single compilation lavel");
	#endif // TIERED
	    // We accept a higher level osr method
	    nmethod* nm = method->lookup_osr_nmethod_for(osr_bci, comp_level, false);
	    if (nm != NULL) return nm;
	    if (method->is_not_osr_compilable()) return NULL;
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!HAS_PENDING_EXCEPTION, "No exception should be present");

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // some prerequisites that are compiler specific
	  if (compiler(comp_level)->is_c2() || compiler(comp_level)->is_shark()) {
	    method->constants()->resolve_string_constants(CHECK_0);
	    // Resolve all classes seen in the signature of the method
	    // we are compiling.
	    methodOopDesc::load_signature_classes(method, CHECK_0);
	  }
	
  {- -------------------------------------------
  (1) 対象が native メソッドの場合, 
      JIT コンパイル中はコードのロードは出来ないので
      念のため NativeLookup::lookup() を呼んで確実にロードされた状態にしておく.
  
      (なお, NativeLookup::lookup() で例外が発生した場合は, その例外を消した後で, ここでリターンする.
      例外を消してしまっているが, どうせインタープリタ側で lookup した時に気づくだろう, という判断っぽい)
      ---------------------------------------- -}

	  // If the method is native, do the lookup in the thread requesting
	  // the compilation. Native lookups can load code, which is not
	  // permitted during compilation.
	  //
	  // Note: A native method implies non-osr compilation which is
	  //       checked with an assertion at the entry of this method.
	  if (method->is_native()) {
	    bool in_base_library;
	    address adr = NativeLookup::lookup(method, in_base_library, THREAD);
	    if (HAS_PENDING_EXCEPTION) {
	      // In case of an exception looking up the method, we just forget
	      // about it. The interpreter will kick-in and throw the exception.
	      method->set_not_compilable(); // implies is_not_osr_compilable()
	      CLEAR_PENDING_EXCEPTION;
	      return NULL;
	    }
	    assert(method->has_native_function(), "must have native code by now");
	  }
	
  {- -------------------------------------------
  (1) JVMTI の RedefineClasses() によって置き換えられていた場合,
      (JIT コンパイルしても意味が無いので) ここでリターン.
      ---------------------------------------- -}

	  // RedefineClasses() has replaced this method; just return
	  if (method->is_old()) {
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) (JVMTI のフック点用の処理)
      (CompiledMethodLoad イベントを投げる必要がある場合, 
      後で nmethod::jmethod_id() の値が必要になるかもしれないが
      JIT コンパイラスレッドにはその計算に必要なロックを取得する権限がないかもしれないので, 
      ここで先に呼び出してプリフェッチしておく)
      ---------------------------------------- -}

	  // JVMTI -- post_compile_event requires jmethod_id() that may require
	  // a lock the compiling thread can not acquire. Prefetch it here.
	  if (JvmtiExport::should_post_compiled_method_load()) {
	    method->jmethod_id();
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // If the compiler is shut off due to code cache flushing or otherwise,
	  // fail out now so blocking compiles dont hang the java thread
	  if (!should_compile_new_jobs() || (UseCodeCacheFlushing && CodeCache::needs_flushing())) {
	    CompilationPolicy::policy()->delay_compilation(method());
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) 以下のどちらかの方法で実際の JIT コンパイル処理を行う.
  
      * 対象が native メソッドの場合, 
        AdapterHandlerLibrary::create_native_wrapper() を呼んで処理を行う.
        (ただし, PreferInterpreterNativeStubs オプションが true の場合は,
         native メソッドの JIT コンパイルは禁止なので, 処理はせずに NULL をリターン)
      * 対象が native メソッドでない場合, 
        CompileBroker::compile_method_base() を呼んで処理を行う.
  
      (ところで, ここの native メソッドのパスでは
      CompileBroker::assign_compile_id() の返値が 0 かどうかかチェックしてないけどいいのかな? #TODO)
      ---------------------------------------- -}

	  // do the compilation
	  if (method->is_native()) {
	    if (!PreferInterpreterNativeStubs) {
	      // Acquire our lock.
	      int compile_id;
	      {
	        MutexLocker locker(MethodCompileQueue_lock, THREAD);
	        compile_id = assign_compile_id(method, standard_entry_bci);
	      }
	      (void) AdapterHandlerLibrary::create_native_wrapper(method, compile_id);
	    } else {
	      return NULL;
	    }
	  } else {
	    compile_method_base(method, osr_bci, comp_level, hot_method, hot_count, comment, CHECK_0);
	  }
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  // return requested nmethod
	  // We accept a higher level osr method
	  return osr_bci  == InvocationEntryBci ? method->code() : method->lookup_osr_nmethod_for(osr_bci, comp_level, false);
	}
	
```


