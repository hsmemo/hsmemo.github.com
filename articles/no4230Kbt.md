---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/init.cpp

### 名前(function name)
```
jint init_globals() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  HandleMark hm;

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  management_init();
	  bytecodes_init();

  {- -------------------------------------------
  (1) ClassLoader を初期化 (See: [here](no7882afy.html) for details)
      ---------------------------------------- -}

	  classLoader_init();

  {- -------------------------------------------
  (1) CodeCache を初期化
      ---------------------------------------- -}

	  codeCache_init();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  VM_Version_init();
	  stubRoutines_init1();
	  jint status = universe_init();  // dependent on codeCache_init and stubRoutines_init
	  if (status != JNI_OK)
	    return status;
	
	  interpreter_init();  // before any methods loaded
	  invocationCounter_init();  // before any methods loaded
	  marksweep_init();
	  accessFlags_init();
	  templateTable_init();
	  InterfaceSupport_init();
	  SharedRuntime::generate_stubs();

  {- -------------------------------------------
  (1) ?? #TODO
      (この中で Klass オブジェクトの生成と初期化を行っている模様.
      Klass オブジェクトのメソッド allocate() で対応する Oop を確保するため, こいつらはあらかじめ作っておく必要がある)
      (Klass オブジェクトの初期化以外に, SystemDictionary の初期化など, universe2_init() の中ではいろんなことを行っている模様)
      ---------------------------------------- -}

	  universe2_init();  // dependent on codeCache_init and stubRoutines_init

  {- -------------------------------------------
  (1) ReferenceProcessor に関する初期化を行う. (See: [here](no289169tf.html) for details)
      ---------------------------------------- -}

	  referenceProcessor_init();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  jni_handles_init();
	#ifndef VM_STRUCTS_KERNEL
	  vmStructs_init();
	#endif // VM_STRUCTS_KERNEL
	
	  vtableStubs_init();
	  InlineCacheBuffer_init();
	  compilerOracle_init();
	  compilationPolicy_init();
	  VMRegImpl::set_regName();
	
	  if (!universe_post_init()) {
	    return JNI_ERR;
	  }
	  javaClasses_init();  // must happen after vtable initialization
	  stubRoutines_init2(); // note: StubRoutines need 2-phase init
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  // Although we'd like to, we can't easily do a heap verify
	  // here because the main thread isn't yet a JavaThread, so
	  // its TLAB may not be made parseable from the usual interfaces.
	  if (VerifyBeforeGC && !UseTLAB &&
	      Universe::heap()->total_collections() >= VerifyGCStartAt) {
	    Universe::heap()->prepare_for_verify();
	    Universe::verify();   // make sure we're starting with a clean slate
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  // All the flags that get adjusted by VM_Version_init and os::init_2
	  // have been set so dump the flags now.
	  if (PrintFlagsFinal) {
	    CommandLineFlags::printFlags();
	  }
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JNI_OK;
	}
	
```


