---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp

### 名前(function name)
```
JVM_ENTRY(void, JVM_Sleep(JNIEnv* env, jclass threadClass, jlong millis))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper)
      ---------------------------------------- -}

	  JVMWrapper("JVM_Sleep");
	
  {- -------------------------------------------
  (1) スリープ時間として負値が指定されていたら IllegalArgumentException.
      ---------------------------------------- -}

	  if (millis < 0) {
	    THROW_MSG(vmSymbols::java_lang_IllegalArgumentException(), "timeout value is negative");
	  }
	
  {- -------------------------------------------
  (1) もし処理対象のスレッドに対して java.lang.Thread.interrupt() が呼ばれていたら, 
      この時点で InterruptedException を出す.
      ---------------------------------------- -}

	  if (Thread::is_interrupted (THREAD, true) && !HAS_PENDING_EXCEPTION) {
	    THROW_MSG(vmSymbols::java_lang_InterruptedException(), "sleep interrupted");
	  }
	
  {- -------------------------------------------
  (1) JavaThreadSleepState で, JavaThread の状態を一時的に SLEEPING に変更
      (ついでに JMM 用の(プロファイル情報の記録)も行っている.
       See: JavaThreadSleepState)
      ---------------------------------------- -}

	  // Save current thread state and restore it at the end of this block.
	  // And set new thread state to SLEEPING.
	  JavaThreadSleepState jtss(thread);
	
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  HS_DTRACE_PROBE1(hotspot, thread__sleep__begin, millis);
	
  {- -------------------------------------------
  (1) 条件に応じて, それぞれ以下の関数を呼び出す.
      * 引数の sleep 時間が 0 で, かつ ConvertSleepToYield オプションが指定されている
        os::yield()
      * 引数の sleep 時間が 0 で, かつ ConvertSleepToYield オプションは指定されていない
        os::sleep()
      * 引数の sleep 時間が 0 ではない
        os::sleep()
  
      なお, 引数の sleep 時間が 0 ではないケースについては, 
      os::sleep() の返り値が OS_INTRPT でかつ pending exception がなければ 
      InterruptedException を出す.
      (See: java.lang.Thread.interrupt())
      ---------------------------------------- -}

	  if (millis == 0) {
	    // When ConvertSleepToYield is on, this matches the classic VM implementation of
	    // JVM_Sleep. Critical for similar threading behaviour (Win32)
	    // It appears that in certain GUI contexts, it may be beneficial to do a short sleep
	    // for SOLARIS
	    if (ConvertSleepToYield) {
	      os::yield();
	    } else {
	      ThreadState old_state = thread->osthread()->get_state();
	      thread->osthread()->set_state(SLEEPING);
	      os::sleep(thread, MinSleepInterval, false);
	      thread->osthread()->set_state(old_state);
	    }
	  } else {
	    ThreadState old_state = thread->osthread()->get_state();
	    thread->osthread()->set_state(SLEEPING);
	    if (os::sleep(thread, millis, true) == OS_INTRPT) {
	      // An asynchronous exception (e.g., ThreadDeathException) could have been thrown on
	      // us while we were sleeping. We do not overwrite those.
	      if (!HAS_PENDING_EXCEPTION) {
	        HS_DTRACE_PROBE1(hotspot, thread__sleep__end,1);
	        // TODO-FIXME: THROW_MSG returns which means we will not call set_state()
	        // to properly restore the thread state.  That's likely wrong.
	        THROW_MSG(vmSymbols::java_lang_InterruptedException(), "sleep interrupted");
	      }
	    }
	    thread->osthread()->set_state(old_state);
	  }

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  HS_DTRACE_PROBE1(hotspot, thread__sleep__end,0);
	JVM_END
	
```


