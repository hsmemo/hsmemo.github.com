---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/interpreterRuntime.cpp

### 名前(function name)
```
IRT_ENTRY(void, InterpreterRuntime::_new(JavaThread* thread, constantPoolOopDesc* pool, int index))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  klassOop k_oop = pool->klass_at(index, CHECK);
	  instanceKlassHandle klass (THREAD, k_oop);
	
  {- -------------------------------------------
  (1) instanceKlass::check_valid_for_instantiation() で new 対象のクラスが妥当かどうかをチェックする
      (インターフェースや abstract クラスに対して instantiate しようとしていたら InstantiationError)
      (See: instanceKlass::check_valid_for_instantiation())
      ---------------------------------------- -}

	  // Make sure we are not instantiating an abstract klass
	  klass->check_valid_for_instantiation(true, CHECK);
	
  {- -------------------------------------------
  (1) instanceKlass::initialize() で, リンク&初期化を(もしまだ行われていなければ)実行する.
      ---------------------------------------- -}

	  // Make sure klass is initialized
	  klass->initialize(CHECK);
	
  {- -------------------------------------------
  (1) instanceKlass::allocate_instance() でメモリの確保を行う.
        (確保した結果は, JavaThread::set_vm_result() で JavaThread 内に格納してリターン)
      ---------------------------------------- -}

	  // At this point the class may not be fully initialized
	  // because of recursive initialization. If it is fully
	  // initialized & has_finalized is not set, we rewrite
	  // it into its fast version (Note: no locking is needed
	  // here since this is an atomic byte write and can be
	  // done more than once).
	  //
	  // Note: In case of classes with has_finalized we don't
	  //       rewrite since that saves us an extra check in
	  //       the fast version which then would call the
	  //       slow version anyway (and do a call back into
	  //       Java).
	  //       If we have a breakpoint, then we don't rewrite
	  //       because the _breakpoint bytecode would be lost.
	  oop obj = klass->allocate_instance(CHECK);
	  thread->set_vm_result(obj);
	IRT_END
	
```


