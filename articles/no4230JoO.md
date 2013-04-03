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
_JNI_IMPORT_OR_EXPORT_ jint JNICALL JNI_CreateJavaVM(JavaVM **vm, void **penv, void *args) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  HS_DTRACE_PROBE3(hotspot_jni, CreateJavaVM__entry, vm, penv, args);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jint result = JNI_ERR;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(CreateJavaVM, jint, (const jint&)result);
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      (ZERO が使用している GCC の __sync_lock_test_and_set() 関数が
       ここで想定しているアトミックな処理をしてくれるとは限らないため, 念のためチェック)
      ---------------------------------------- -}

	  // We're about to use Atomic::xchg for synchronization.  Some Zero
	  // platforms use the GCC builtin __sync_lock_test_and_set for this,
	  // but __sync_lock_test_and_set is not guaranteed to do what we want
	  // on all architectures.  So we check it works before relying on it.
	#if defined(ZERO) && defined(ASSERT)
	  {
	    jint a = 0xcafebabe;
	    jint b = Atomic::xchg(0xdeadbeef, &a);
	    void *c = &a;
	    void *d = Atomic::xchg_ptr(&b, &c);
	    assert(a == (jint) 0xdeadbeef && b == (jint) 0xcafebabe, "Atomic::xchg() works");
	    assert(c == &b && d == &a, "Atomic::xchg_ptr() works");
	  }
	#endif // ZERO && ASSERT
	
  {- -------------------------------------------
  (1) 複数のスレッドが同時に JNI_CreateJavaVM を呼び出していないか検査する.
  
      (1プロセス内に最大1つしか HotSpot を作らせないようにするため)
  
      (ちなみに, 初期化プロセスが "point of no return" という点まで到達すると, 
      再初期化不可能な static なデータ構造を生成し終わるため, もう HotSpot は作れなくなる.
      この検査は, そこに到達する前に別のスレッドが JNI_CreateJavaVM() を呼んだ場合用のチェック)
      ---------------------------------------- -}

	  // At the moment it's only possible to have one Java VM,
	  // since some of the runtime state is in global variables.
	
	  // We cannot use our mutex locks here, since they only work on
	  // Threads. We do an atomic compare and exchange to ensure only
	  // one thread can call this method at a time
	
	  // We use Atomic::xchg rather than Atomic::add/dec since on some platforms
	  // the add/dec implementations are dependent on whether we are running
	  // on a multiprocessor, and at this stage of initialization the os::is_MP
	  // function used to determine this will always return false. Atomic::xchg
	  // does not have this problem.
	  if (Atomic::xchg(1, &vm_created) == 1) {
	    return JNI_ERR;   // already created, or create attempt in progress
	  }
	  if (Atomic::xchg(0, &safe_to_recreate_vm) == 0) {
	    return JNI_ERR;  // someone tried and failed and retry not allowed.
	  }
	
	  assert(vm_created == 1, "vm_created is true during the creation");
	
  {- -------------------------------------------
  (1) JNI_CreateJavaVM() は, 失敗しても引数などを変えて呼び直せば成功することもあるが, 
      ある一定以上まで処理が進んでしまってから失敗するとやり直せない.
    
      そこで, safe_to_recreate_vm と can_try_again という変数を使って以下のように制御している.
  
      (まず, JNI_CreateJavaVM() を呼んだスレッドは safe_to_recreate_vm を 1 から 0 にする.
       その後, JNI_CreateJavaVM() が失敗した場合は can_try_again の値を確認し, それが 1 なら
       safe_to_recreate_vm を 1 に戻してリターンする.
  
       もし, やり直せないところまで進んでしまっていたら can_try_again が 0 で帰ってくるので
       safe_to_recreate_vm も 0 のままになり, 次の JNI_CreateJavaVM() を呼んだスレッドにもやり直せないことが伝わる)
      ---------------------------------------- -}

	  /**
	   * Certain errors during initialization are recoverable and do not
	   * prevent this method from being called again at a later time
	   * (perhaps with different arguments).  However, at a certain
	   * point during initialization if an error occurs we cannot allow
	   * this function to be called again (or it will crash).  In those
	   * situations, the 'canTryAgain' flag is set to false, which atomically
	   * sets safe_to_recreate_vm to 1, such that any new call to
	   * JNI_CreateJavaVM will immediately fail using the above logic.
	   */
	  bool can_try_again = true;
	
  {- -------------------------------------------
  (1) Threads::create_vm() を呼び出す (この中で初期化処理の大半が行われる).
      ---------------------------------------- -}

	  result = Threads::create_vm((JavaVMInitArgs*) args, &can_try_again);

  {- -------------------------------------------
  (1) Threads::create_vm() が成功した場合は, 以下の処理を行う.
      ---------------------------------------- -}

	  if (result == JNI_OK) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    JavaThread *thread = JavaThread::current();
	    /* thread is thread_in_vm here */

    {- -------------------------------------------
  (1.1) 引数の JavaVM* と JNIEnv* に, 適切な JavaVM 構造体とJNI インタフェースポインタを設定する.
        ---------------------------------------- -}

	    *vm = (JavaVM *)(&main_vm);
	    *(JNIEnv**)penv = thread->jni_environment();
	
    {- -------------------------------------------
  (1.1) RuntimeService のタイマーを開始させる.
        ---------------------------------------- -}

	    // Tracks the time application was running before GC
	    RuntimeService::record_application_start();
	
    {- -------------------------------------------
  (1.1) (JVMTI のフック点)
        ---------------------------------------- -}

	    // Notify JVMTI
	    if (JvmtiExport::should_post_thread_life()) {
	       JvmtiExport::post_thread_start(thread);
	    }

    {- -------------------------------------------
  (1.1) (デバッグ用の処理) (NOT_PRODUCT 時にのみ実行)
        ---------------------------------------- -}

	    // Check if we should compile all classes on bootclasspath
	    NOT_PRODUCT(if (CompileTheWorld) ClassLoader::compile_the_world();)

    {- -------------------------------------------
  (1.1) カレントスレッドの JavaThreadState を _thread_in_vm から _thread_in_native に変更.
        (JVM_ENTRY() マクロを使っていないので明示的に変更しないといけない)
        ---------------------------------------- -}

	    // Since this is not a JVM_ENTRY we have to set the thread state manually before leaving.
	    ThreadStateTransition::transition_and_fence(thread, _thread_in_vm, _thread_in_native);

  {- -------------------------------------------
  (1) Threads::create_vm() が失敗した場合は, 以下の処理を行う.
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) can_try_again の値が 1 なら safe_to_recreate_vm を 1 に戻す (上の説明も参照)
        ---------------------------------------- -}

	    if (can_try_again) {
	      // reset safe_to_recreate_vm to 1 so that retrial would be possible
	      safe_to_recreate_vm = 1;
	    }
	
    {- -------------------------------------------
  (1.1) 引数の JavaVM* と JNIEnv* には NULL をセットする.
        ---------------------------------------- -}

	    // Creation failed. We must reset vm_created
	    *vm = 0;
	    *(JNIEnv**)penv = 0;

    {- -------------------------------------------
  (1.1) (メモリバリアを張って) vm_created を 0 に戻す.
        ---------------------------------------- -}

	    // reset vm_created last to avoid race condition. Use OrderAccess to
	    // control both compiler and architectural-based reordering.
	    OrderAccess::release_store(&vm_created, 0);
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (NOT_PRODUCT 時にのみ実行)
      ---------------------------------------- -}

	  NOT_PRODUCT(test_error_handler(ErrorHandlerTest));

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return result;
	}
	
```


