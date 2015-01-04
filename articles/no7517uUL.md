---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/instanceKlass.cpp

### 名前(function name)
```
void instanceKlass::initialize_impl(instanceKlassHandle this_oop, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (初期化前にリンクされていないとまずいので) 
      instanceKlass::link_class() を呼んで確実にリンク処理までは終わらせておく.
      ---------------------------------------- -}

	  // Make sure klass is linked (verified) before initialization
	  // A class could already be verified, since it has been reflected upon.
	  this_oop->link_class(CHECK);
	
  {- -------------------------------------------
  (1) (DTrace のフック点) (class__initialization__required)
      ---------------------------------------- -}

	  DTRACE_CLASSINIT_PROBE(required, instanceKlass::cast(this_oop()), -1);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  bool wait = false;
	
  {- -------------------------------------------
  (1) コメントによると, 以下の処理ステップについては "JVM book page 47" を参照, とのこと.
      (現行の Java 仮想マシン仕様の "5.5 Initialization" の内容だと思われる(?))
      (なお, HotSpot 内では, 仕様における initialization lock ("LC") として, 
       初期化対象クラスのモニターを用いている (以下の処理参照))
      ---------------------------------------- -}

	  // refer to the JVM book page 47 for description of steps

  {- -------------------------------------------
  (1) (以下のブロック内の処理は, initialization lock ("LC") で排他した状態で行う)
      (Java 仮想マシン仕様の "5.5 Initialization" における Step 1 の内容)
      ---------------------------------------- -}

	  // Step 1
	  { ObjectLocker ol(this_oop, THREAD);
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    Thread *self = THREAD; // it's passed the current thread
	
    {- -------------------------------------------
  (1.1) もし他のスレッドによって初期化中であれば, 
        初期化が完了するまでここで待機する.
        (Java 仮想マシン仕様の "5.5 Initialization" における Step 2 の内容)
  
        (なおコメントによると, 
         waitUninterruptibly() の代わりに wait() で待つと
         InterruptedException が出る恐れがあり, 
         これは link/symbol resolution では予期されていない例外なので酷いことになる, とのこと)
        ---------------------------------------- -}

	    // Step 2
	    // If we were to use wait() instead of waitInterruptibly() then
	    // we might end up throwing IE from link/symbol resolution sites
	    // that aren't expected to throw.  This would wreak havoc.  See 6320309.
	    while(this_oop->is_being_initialized() && !this_oop->is_reentrant_initialization(self)) {
	        wait = true;
	      ol.waitUninterruptibly(CHECK);
	    }
	
    {- -------------------------------------------
  (1.1) もしカレントスレッドによって初期化中であれば, 
        これ以上処理をする必要はないので, ここでリターン.
        (Java 仮想マシン仕様の "5.5 Initialization" における Step 3 の内容)
  
        なお, (DTrace のフック点) でもある (class__initialization__recursive)
        ---------------------------------------- -}

	    // Step 3
	    if (this_oop->is_being_initialized() && this_oop->is_reentrant_initialization(self)) {
	      DTRACE_CLASSINIT_PROBE_WAIT(recursive, instanceKlass::cast(this_oop()), -1,wait);
	      return;
	    }
	
    {- -------------------------------------------
  (1.1) もし既に初期化済みであれば, 
        これ以上処理をする必要はないので, ここでリターン.
        (Java 仮想マシン仕様の "5.5 Initialization" における Step 4 の内容)
  
        なお, (DTrace のフック点) でもある (class__initialization__concurrent)
        ---------------------------------------- -}

	    // Step 4
	    if (this_oop->is_initialized()) {
	      DTRACE_CLASSINIT_PROBE_WAIT(concurrent, instanceKlass::cast(this_oop()), -1,wait);
	      return;
	    }
	
    {- -------------------------------------------
  (1.1) もし対象クラスが初期化中にエラーが起こってしまった状態であれば
        ここでもう一度初期化処理をしても意味はないので, 
        NoClassDefFoundError を出して終了.
        (Java 仮想マシン仕様の "5.5 Initialization" における Step 5 の内容)
  
        なお, (DTrace のフック点) でもある (class__initialization__erroneous)
        ---------------------------------------- -}

	    // Step 5
	    if (this_oop->is_in_error_state()) {
	      DTRACE_CLASSINIT_PROBE_WAIT(erroneous, instanceKlass::cast(this_oop()), -1,wait);
	      ResourceMark rm(THREAD);
	      const char* desc = "Could not initialize class ";
	      const char* className = this_oop->external_name();
	      size_t msglen = strlen(desc) + strlen(className) + 1;
	      char* message = NEW_RESOURCE_ARRAY(char, msglen);
	      if (NULL == message) {
	        // Out of memory: can't create detailed error message
	        THROW_MSG(vmSymbols::java_lang_NoClassDefFoundError(), className);
	      } else {
	        jio_snprintf(message, msglen, "%s%s", desc, className);
	        THROW_MSG(vmSymbols::java_lang_NoClassDefFoundError(), message);
	      }
	    }
	
    {- -------------------------------------------
  (1.1) 以上のどれでもなければ, 対象クラスはカレントスレッドが初期化することになるので, 
        instanceKlass::set_init_state() 及び instanceKlass::set_init_thread() を呼んで
        そのことを対象クラス内に記録しておく
        (ここまでが initialization lock ("LC") で排他した処理)
  
        (Java 仮想マシン仕様の "5.5 Initialization" における Step 6 の内容)
        ---------------------------------------- -}

	    // Step 6
	    this_oop->set_init_state(being_initialized);
	    this_oop->set_init_thread(self);
	  }
	
  {- -------------------------------------------
  (1) 対象がインターフェースではなくクラスの場合, 
      スーパークラスが初期化されてなければ, 
      Klass::initialize() を再帰的に呼び出してスーパークラスを初期化する.
  
      (もし初期化でエラーが出た場合は, 
      instanceKlass::set_initialization_state_and_notify() を呼んで
      クラスの初期化状態を initialization_error に変更し, 待機している他スレッドを起床させた後, 
      例外を投げ直す.
      なお, (DTrace のフック点) でもある (class__initialization__super__failed))
  
      (Java 仮想マシン仕様の "5.5 Initialization" における Step 7 の内容)
      ---------------------------------------- -}

	  // Step 7
	  klassOop super_klass = this_oop->super();
	  if (super_klass != NULL && !this_oop->is_interface() && Klass::cast(super_klass)->should_be_initialized()) {
	    Klass::cast(super_klass)->initialize(THREAD);
	
	    if (HAS_PENDING_EXCEPTION) {
	      Handle e(THREAD, PENDING_EXCEPTION);
	      CLEAR_PENDING_EXCEPTION;
	      {
	        EXCEPTION_MARK;
	        this_oop->set_initialization_state_and_notify(initialization_error, THREAD); // Locks object, set state, and notify all waiting threads
	        CLEAR_PENDING_EXCEPTION;   // ignore any exception thrown, superclass initialization error is thrown below
	      }
	      DTRACE_CLASSINIT_PROBE_WAIT(super__failed, instanceKlass::cast(this_oop()), -1,wait);
	      THROW_OOP(e());
	    }
	  }
	
  {- -------------------------------------------
  (1) instanceKlass::call_class_initializer() を呼んで, 
      対象クラスの static 初期化子 (= <clinit>メソッド) を実行する
      (Java 仮想マシン仕様の "5.5 Initialization" における Step 9 の内容)
      (コメントでは Step 8 になっているので, 以前の仕様では Step 8 だった?)
  
      なお, (DTrace のフック点) でもある (class__initialization__clinit)
      また, (プロファイル情報の記録) も行っている (See: PerfClassTraceTime)
      ---------------------------------------- -}

	  // Step 8
	  {
	    assert(THREAD->is_Java_thread(), "non-JavaThread in initialize_impl");
	    JavaThread* jt = (JavaThread*)THREAD;
	    DTRACE_CLASSINIT_PROBE_WAIT(clinit, instanceKlass::cast(this_oop()), -1,wait);
	    // Timer includes any side effects of class initialization (resolution,
	    // etc), but not recursive entry into call_class_initializer().
	    PerfClassTraceTime timer(ClassLoader::perf_class_init_time(),
	                             ClassLoader::perf_class_init_selftime(),
	                             ClassLoader::perf_classes_inited(),
	                             jt->get_thread_stat()->perf_recursion_counts_addr(),
	                             jt->get_thread_stat()->perf_timers_addr(),
	                             PerfClassTraceTime::CLASS_CLINIT);
	    this_oop->call_class_initializer(THREAD);
	  }
	
  {- -------------------------------------------
  (1) もし初期化処理が正常に終了した場合は (= 例外が出ていなければ), 
      instanceKlass::set_initialization_state_and_notify() を呼んで
      クラスの初期化状態を fully_initialized に変更し
      待機している他スレッドを起床させる.
  
      (Java 仮想マシン仕様の "5.5 Initialization" における Step 10 の内容)
      (コメントでは Step 9 になっているので, 以前の仕様では Step 9 だった?)
      ---------------------------------------- -}

	  // Step 9
	  if (!HAS_PENDING_EXCEPTION) {
	    this_oop->set_initialization_state_and_notify(fully_initialized, CHECK);
	    { ResourceMark rm(THREAD);
	      debug_only(this_oop->vtable()->verify(tty, true);)
	    }
	  }

  {- -------------------------------------------
  (1) もし初期化処理で例外が出ていた場合は, 
      instanceKlass::set_initialization_state_and_notify() を呼んで
      クラスの初期化状態を initialization_error に変更し, 待機している他スレッドを起床させた後, 
      例外を投げ直す.
  
      (Java 仮想マシン仕様の "5.5 Initialization" における Step 11 及び Step 12 の内容)
      (コメントでは Step 10 and 11 になっているので, 以前の仕様では Step 10 と 11 だった?)
  
      なお, (DTrace のフック点) でもある (class__initialization__error)
      ---------------------------------------- -}

	  else {
	    // Step 10 and 11
	    Handle e(THREAD, PENDING_EXCEPTION);
	    CLEAR_PENDING_EXCEPTION;
	    {
	      EXCEPTION_MARK;
	      this_oop->set_initialization_state_and_notify(initialization_error, THREAD);
	      CLEAR_PENDING_EXCEPTION;   // ignore any exception thrown, class initialization error is thrown below
	    }
	    DTRACE_CLASSINIT_PROBE_WAIT(error, instanceKlass::cast(this_oop()), -1,wait);
	    if (e->is_a(SystemDictionary::Error_klass())) {
	      THROW_OOP(e());
	    } else {
	      JavaCallArguments args(e);
	      THROW_ARG(vmSymbols::java_lang_ExceptionInInitializerError(),
	                vmSymbols::throwable_void_signature(),
	                &args);
	    }
	  }

  {- -------------------------------------------
  (1) (DTrace のフック点) (class__initialization__end)
      ---------------------------------------- -}

	  DTRACE_CLASSINIT_PROBE_WAIT(end, instanceKlass::cast(this_oop()), -1,wait);
	}
	
```


