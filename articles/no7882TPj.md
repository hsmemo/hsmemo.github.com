---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp

### 名前(function name)
```
  static inline void transition_from_native(JavaThread *thread, JavaThreadState to) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert((to & 1) == 0, "odd numbers are transitions states");
	    assert(thread->thread_state() == _thread_in_native, "coming from wrong thread state");

  {- -------------------------------------------
  (1) 処理対象のスレッドの JavaThreadState を _thread_in_native_trans に変更する.
  
      なお, MP 環境の場合には
      VM Thread 側への visibility (sequential consistency) を保証する必要があるので, 
      変更後に以下のどちらかを行っている (See: SafepointSynchronize::begin()).
      * UseMembar オプションが指定されている場合:
        OrderAccess::fence() でメモリバリアを張る.
      * そうではない場合: 
        InterfaceSupport::serialize_memory() で serialize page への書き込みを行う.
      ---------------------------------------- -}

	    // Change to transition state (assumes total store ordering!  -Urs)
	    thread->set_thread_state(_thread_in_native_trans);
	
	    // Make sure new state is seen by GC thread
	    if (os::is_MP()) {
	      if (UseMembar) {
	        // Force a fence between the write above and read below
	        OrderAccess::fence();
	      } else {
	        // Must use this rather than serialization page in particular on Windows
	        InterfaceSupport::serialize_memory(thread);
	      }
	    }
	
  {- -------------------------------------------
  (1) 以下の 2つを確認し, もしこういった状況が発生していたら, 
      JavaThread::check_safepoint_and_suspend_for_native_trans() を呼んで待機する.
      * Safepoint 処理が開始されているかどうか  (= SafepointSynchronize::do_call_back()) 
      * カレントスレッドがサスペンドされているかどうか  (= thread->is_suspend_after_native())
  
      (なお, asynchronous exception はチェックしない. #TODO)
  
  
      (なお, CHECK_UNHANDLED_OOPS が #define されている場合には, 
       JavaThread::check_safepoint_and_suspend_for_native_trans() を呼んだ場合, 
       unhandled oop をクリアする処理も行っている.)
      ---------------------------------------- -}

	    // We never install asynchronous exceptions when coming (back) in
	    // to the runtime from native code because the runtime is not set
	    // up to handle exceptions floating around at arbitrary points.
	    if (SafepointSynchronize::do_call_back() || thread->is_suspend_after_native()) {
	      JavaThread::check_safepoint_and_suspend_for_native_trans(thread);
	
	      // Clear unhandled oops anywhere where we could block, even if we don't.
	      CHECK_UNHANDLED_OOPS_ONLY(thread->clear_unhandled_oops();)
	    }
	
  {- -------------------------------------------
  (1) 処理対象のスレッドの JavaThreadState を, to 引数の値に変更する.
      ---------------------------------------- -}

	    thread->set_thread_state(to);
	  }
	
```


