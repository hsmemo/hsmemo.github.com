---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.cpp

### 名前(function name)
```
int os::sleep(Thread* thread, jlong millis, bool interruptible) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(thread == Thread::current(),  "thread consistency check");
	
  {- -------------------------------------------
  (1) os::PlatformEvent::reset() を呼んで, 
      処理対象のスレッドの _SleepEvent フィールド (ParkEvent オブジェクト) をリセットしておく.
    
      (ついでに, OrderAccess::fence() でメモリバリアも張っておく.
       以降の park() 操作での load/store がこの初期化操作の store を追い抜くのは禁止.)
      ---------------------------------------- -}

	  ParkEvent * const slp = thread->_SleepEvent ;
	  slp->reset() ;
	  OrderAccess::fence() ;
	
  {- -------------------------------------------
  (1) (以下の処理は, java.lang.Thread.interrupt() による割り込みを
       許すかどうか(= 引数の interruptible が true か否か)に応じて, 2通りに分岐)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下が, java.lang.Thread.interrupt() による割り込みを許す場合の処理)
  
      ThreadBlockInVM 及び OSThreadWaitState で JavaThread/OSThread の状態を変更した後, 
      _SleepEvent フィールドに対して os::PlatformEvent::park() を呼ぶことで眠りにつく. 
      (予定より早く起きることもあり得るので, 目が覚める度に javaTimeNanos() で時間を確認し, 
       ちゃんと指定時間分だけ経過するまで os::PlatformEvent::park() の呼び出しを繰り返す.)
  
      (なお, 寝ている間に java.lang.Thread.suspend() で suspend 状態にされているかもしれないので, 
       目が覚める度に JavaThread::check_and_wait_while_suspended() でチェックを行っている.
       もし suspend されていれば, この中で resume されるまで待機する
       (See: java.lang.Thread.suspend()))
      
      (また, 寝ている間に java.lang.Thread.interrupt() で割り込まれたのかもしれないので, 
       毎回 os::is_interrupted() でチェックを行っている.
       もし割り込まれていれば, その時点でリターンする (OS_INTRPT をリターン).
       (See: java.lang.Thread.interrupt()))
  
  
      #TODO  JavaThread::set_suspend_equivalent() はどういう意味がある??
      ---------------------------------------- -}

	  if (interruptible) {
	    jlong prevtime = javaTimeNanos();
	
	    for (;;) {
	      if (os::is_interrupted(thread, true)) {
	        return OS_INTRPT;
	      }
	
	      jlong newtime = javaTimeNanos();
	
	      if (newtime - prevtime < 0) {
	        // time moving backwards, should only happen if no monotonic clock
	        // not a guarantee() because JVM should not abort on kernel/glibc bugs
	        assert(!Linux::supports_monotonic_clock(), "time moving backwards");
	      } else {
	        millis -= (newtime - prevtime) / NANOSECS_PER_MILLISECS;
	      }
	
	      if(millis <= 0) {
	        return OS_OK;
	      }
	
	      prevtime = newtime;
	
	      {
	        assert(thread->is_Java_thread(), "sanity check");
	        JavaThread *jt = (JavaThread *) thread;
	        ThreadBlockInVM tbivm(jt);
	        OSThreadWaitState osts(jt->osthread(), false /* not Object.wait() */);
	
	        jt->set_suspend_equivalent();
	        // cleared by handle_special_suspend_equivalent_condition() or
	        // java_suspend_self() via check_and_wait_while_suspended()
	
	        slp->park(millis);
	
	        // were we externally suspended while we were waiting?
	        jt->check_and_wait_while_suspended();
	      }
	    }

  {- -------------------------------------------
  (1) (以下が, java.lang.Thread.interrupt() による割り込みを許さない場合の処理)
  
      OSThreadWaitState で OSThread の状態を変更した後, 
      _SleepEvent フィールドに対して os::PlatformEvent::park() を呼ぶことで眠りにつく. 
      (予定より早く起きることもあり得るので, 目が覚める度に javaTimeNanos() で時間を確認し, 
       ちゃんと指定時間分だけ経過するまで os::PlatformEvent::park() の呼び出しを繰り返す.)
      ---------------------------------------- -}

	  } else {
	    OSThreadWaitState osts(thread->osthread(), false /* not Object.wait() */);
	    jlong prevtime = javaTimeNanos();
	
	    for (;;) {
	      // It'd be nice to avoid the back-to-back javaTimeNanos() calls on
	      // the 1st iteration ...
	      jlong newtime = javaTimeNanos();
	
	      if (newtime - prevtime < 0) {
	        // time moving backwards, should only happen if no monotonic clock
	        // not a guarantee() because JVM should not abort on kernel/glibc bugs
	        assert(!Linux::supports_monotonic_clock(), "time moving backwards");
	      } else {
	        millis -= (newtime - prevtime) / NANOSECS_PER_MILLISECS;
	      }
	
	      if(millis <= 0) break ;
	
	      prevtime = newtime;
	      slp->park(millis);
	    }
	    return OS_OK ;
	  }
	}
	
```


