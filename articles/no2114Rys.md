---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp
### 説明(description)

```
// Dump thread info for the specified threads.
// It returns an array of ThreadInfo objects. Each element is the ThreadInfo
// for the thread ID specified in the corresponding entry in
// the given array of thread IDs; or NULL if the thread does not exist
// or has terminated.
//
// Input parameter:
//    ids - array of thread IDs; NULL indicates all live threads
//    locked_monitors - if true, dump locked object monitors
//    locked_synchronizers - if true, dump locked JSR-166 synchronizers
//
```

### 名前(function name)
```
JVM_ENTRY(jobjectArray, jmm_DumpThreads(JNIEnv *env, jlongArray thread_ids, jboolean locked_monitors, jboolean locked_synchronizers))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);
	
  {- -------------------------------------------
  (1) JDK のバージョンが 1.6 以上の場合には
      java.util.concurrent.locks.AbstractOwnableSynchronizer クラスを使用するので, 
      もしまだロードしていなければロードしておく.
      ---------------------------------------- -}

	  if (JDK_Version::is_gte_jdk16x_version()) {
	    // make sure the AbstractOwnableSynchronizer klass is loaded before taking thread snapshots
	    java_util_concurrent_locks_AbstractOwnableSynchronizer::initialize(CHECK_NULL);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  typeArrayOop ta = typeArrayOop(JNIHandles::resolve(thread_ids));
	  int num_threads = (ta != NULL ? ta->length() : 0);
	  typeArrayHandle ids_ah(THREAD, ta);
	
  {- -------------------------------------------
  (1) 対象のスレッドの情報を ThreadSnapshot オブジェクトとして取得する.
      (取得した結果は ThreadDumpResult オブジェクト (以下の dump_result) に格納される)
      
      なお, 取得方法は引数に応じて 2通りある.
      * 情報取得対象のスレッドが指定されている場合:
        do_thread_dump() で取得.
      * 〃 が指定されていない場合:
        VM_ThreadDump::doit() で取得.
      (といっても, do_thread_dump() も, 内部では VM_ThreadDump::doit() を呼び出すが...)
    
      (なお, 情報取得対象のスレッドが指定されている場合には, 
       validate_thread_id_array() でその指定をチェックしている.
       不正な値が指定されていた場合は IllegalArgumentException.)
      ---------------------------------------- -}

	  ThreadDumpResult dump_result(num_threads);  // can safepoint
	
	  if (ids_ah() != NULL) {
	
	    // validate the thread id array
	    validate_thread_id_array(ids_ah, CHECK_NULL);
	
	    // obtain thread dump of a specific list of threads
	    do_thread_dump(&dump_result,
	                   ids_ah,
	                   num_threads,
	                   -1, /* entire stack */
	                   (locked_monitors ? true : false),      /* with locked monitors */
	                   (locked_synchronizers ? true : false), /* with locked synchronizers */
	                   CHECK_NULL);
	  } else {
	    // obtain thread dump of all threads
	    VM_ThreadDump op(&dump_result,
	                     -1, /* entire stack */
	                     (locked_monitors ? true : false),     /* with locked monitors */
	                     (locked_synchronizers ? true : false) /* with locked synchronizers */);
	    VMThread::execute(&op);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int num_snapshots = dump_result.num_snapshots();
	
  {- -------------------------------------------
  (1) 返値としてリターンするための ThreadInfo 配列を生成する.
      ---------------------------------------- -}

	  // create the result ThreadInfo[] object
	  klassOop k = Management::java_lang_management_ThreadInfo_klass(CHECK_NULL);
	  instanceKlassHandle ik (THREAD, k);
	  objArrayOop r = oopFactory::new_objArray(ik(), num_snapshots, CHECK_NULL);
	  objArrayHandle result_h(THREAD, r);
	
  {- -------------------------------------------
  (1) (以下の for 文の中で
      生成した ThreadSnapshot オブジェクトから ThreadInfo オブジェクトを生成し, 
      引数で渡された配列に詰めていく)
      ---------------------------------------- -}

	  int index = 0;
	  for (ThreadSnapshot* ts = dump_result.snapshots(); ts != NULL; ts = ts->next(), index++) {

    {- -------------------------------------------
  (1.1) 該当するスレッドがない要素については, NULL を詰めておく.
        ---------------------------------------- -}

	    if (ts->threadObj() == NULL) {
	     // if the thread does not exist or now it is terminated, set threadinfo to NULL
	      result_h->obj_at_put(index, NULL);
	      continue;
	    }
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    ThreadStackTrace* stacktrace = ts->get_stack_trace();
	    assert(stacktrace != NULL, "Must have a stack trace dumped");
	
	    // Create Object[] filled with locked monitors
	    // Create int[] filled with the stack depth where a monitor was locked
	    int num_frames = stacktrace->get_stack_depth();
	    int num_locked_monitors = stacktrace->num_jni_locked_monitors();
	
	    // Count the total number of locked monitors
	    for (int i = 0; i < num_frames; i++) {
	      StackFrameInfo* frame = stacktrace->stack_frame_at(i);
	      num_locked_monitors += frame->num_locked_monitors();
	    }
	
	    objArrayHandle monitors_array;
	    typeArrayHandle depths_array;
	    objArrayHandle synchronizers_array;
	
    {- -------------------------------------------
  (1.1) もし引数で「スレッドによって現在ロックされているオブジェクトモニター」の情報も集めるように指示されていれば, 
        StackFrameInfo::locked_monitors() を呼んで
        ロックしているモニターを示す java.lang.management.MonitorInfo オブジェクトを取得する.
        (取得した MonitorInfo オブジェクトは, 配列(以下の monitors_array)中に詰めておく)
    
        ついでに, StackFrameInfo::jni_locked_monitors() も呼んで
        ネイティブメソッド中でロックされているモニターについても
        java.lang.management.MonitorInfo オブジェクトを取得する.
        (そして同様に, 配列(以下の monitors_array)中に詰める)
        ---------------------------------------- -}

	    if (locked_monitors) {
	      // Constructs Object[] and int[] to contain the object monitor and the stack depth
	      // where the thread locked it
	      objArrayOop array = oopFactory::new_objArray(SystemDictionary::Object_klass(), num_locked_monitors, CHECK_NULL);
	      objArrayHandle mh(THREAD, array);
	      monitors_array = mh;
	
	      typeArrayOop tarray = oopFactory::new_typeArray(T_INT, num_locked_monitors, CHECK_NULL);
	      typeArrayHandle dh(THREAD, tarray);
	      depths_array = dh;
	
	      int count = 0;
	      int j = 0;
	      for (int depth = 0; depth < num_frames; depth++) {
	        StackFrameInfo* frame = stacktrace->stack_frame_at(depth);
	        int len = frame->num_locked_monitors();
	        GrowableArray<oop>* locked_monitors = frame->locked_monitors();
	        for (j = 0; j < len; j++) {
	          oop monitor = locked_monitors->at(j);
	          assert(monitor != NULL && monitor->is_instance(), "must be a Java object");
	          monitors_array->obj_at_put(count, monitor);
	          depths_array->int_at_put(count, depth);
	          count++;
	        }
	      }
	
	      GrowableArray<oop>* jni_locked_monitors = stacktrace->jni_locked_monitors();
	      for (j = 0; j < jni_locked_monitors->length(); j++) {
	        oop object = jni_locked_monitors->at(j);
	        assert(object != NULL && object->is_instance(), "must be a Java object");
	        monitors_array->obj_at_put(count, object);
	        // Monitor locked via JNI MonitorEnter call doesn't have stack depth info
	        depths_array->int_at_put(count, -1);
	        count++;
	      }
	      assert(count == num_locked_monitors, "number of locked monitors doesn't match");
	    }
	
    {- -------------------------------------------
  (1.1) もし引数で「スレッドによって現在ロックされている所有可能なシンクロナイザ」の情報も集めるように指示されていれば, 
        ThreadSnapshot::get_concurrent_locks() を呼んで
        ロックしているシンクロナイザを示す ThreadConcurrentLocks オブジェクトを取得する.
    
        その後, ThreadConcurrentLocks::owned_locks() を呼んで, 
        ThreadConcurrentLocks オブジェクトから
        java.lang.management.LockInfo オブジェクトを取り出し, 
        配列(以下の synchronizers_array)中に詰めておく.
        ---------------------------------------- -}

	    if (locked_synchronizers) {
	      // Create Object[] filled with locked JSR-166 synchronizers
	      assert(ts->threadObj() != NULL, "Must be a valid JavaThread");
	      ThreadConcurrentLocks* tcl = ts->get_concurrent_locks();
	      GrowableArray<instanceOop>* locks = (tcl != NULL ? tcl->owned_locks() : NULL);
	      int num_locked_synchronizers = (locks != NULL ? locks->length() : 0);
	
	      objArrayOop array = oopFactory::new_objArray(SystemDictionary::Object_klass(), num_locked_synchronizers, CHECK_NULL);
	      objArrayHandle sh(THREAD, array);
	      synchronizers_array = sh;
	
	      for (int k = 0; k < num_locked_synchronizers; k++) {
	        synchronizers_array->obj_at_put(k, locks->at(k));
	      }
	    }
	
    {- -------------------------------------------
  (1.1) 以上で取得した値を元に, 
        Management::create_thread_info_instance() で ThreadInfo オブジェクトを生成し, 
        返値としてリターンする配列(result_h)に格納する.
        ---------------------------------------- -}

	    // Create java.lang.management.ThreadInfo object
	    instanceOop info_obj = Management::create_thread_info_instance(ts,
	                                                                   monitors_array,
	                                                                   depths_array,
	                                                                   synchronizers_array,
	                                                                   CHECK_NULL);
	    result_h->obj_at_put(index, info_obj);
	  }
	
  {- -------------------------------------------
  (1) 結果を JNIHandle 化してリターン
      ---------------------------------------- -}

	  return (jobjectArray) JNIHandles::make_local(env, result_h());
	JVM_END
	
```


