---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp

### 名前(function name)
```
int os::sleep(Thread* thread, jlong ms, bool interruptable) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 指定されたスリープ時間が MAXDWORD を超えている場合には, 
      MAXDWORD 分ずつに分けてスリープ処理を行う.
      (この関数自身を再帰呼び出しし, MAXDWORD 時間だけのスリープ処理を行う.
       この処理を, 残りのスリープ時間が MAXDWORD 以下になるまで繰り返す.
  
       ただし, 途中で java.lang.Thread.interrupt() によって
       割り込まれた場合(= 返値が OS_TIMEOUT ではなかった場合) には
       その時点で処理を終えてリターン)
      ---------------------------------------- -}

	  jlong limit = (jlong) MAXDWORD;
	
	  while(ms > limit) {
	    int res;
	    if ((res = sleep(thread, limit, interruptable)) != OS_TIMEOUT)
	      return res;
	    ms -= limit;
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(thread == Thread::current(),  "thread consistency check");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  OSThread* osthread = thread->osthread();

  {- -------------------------------------------
  (1) OSThreadWaitState で OSThread の状態を変更しておく.
      ---------------------------------------- -}

	  OSThreadWaitState osts(osthread, false /* not Object.wait() */);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int result;

  {- -------------------------------------------
  (1) (以下の処理は, java.lang.Thread.interrupt() による割り込みを
       許すかどうか(= 引数の interruptible が true か否か)に応じて, 2通りに分岐)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下が, java.lang.Thread.interrupt() による割り込みを許す場合の処理)
  
      ThreadBlockInVM で JavaThread の状態を変更した後, 
      WaitForMultipleObjects() で眠りにつく.
    
      返値(以下の result)は, 
      WaitForMultipleObjects() の返値が WAIT_TIMEOUT でない場合には, 
      (途中で割り込まれたということなので) OS_INTRPT, 
      そうでない場合には OS_TIMEOUT, とする.
    
      (なお, 寝ている間に java.lang.Thread.suspend() で suspend 状態にされているかもしれないので, 
       目が覚めた後に JavaThread::check_and_wait_while_suspended() でチェックを行っている.
       もし suspend されていれば, この中で resume されるまで待機する
       (See: java.lang.Thread.suspend()))
  
  
      #TODO  JavaThread::set_suspend_equivalent() はどういう意味がある??
      ---------------------------------------- -}

	  if (interruptable) {
	    assert(thread->is_Java_thread(), "must be java thread");
	    JavaThread *jt = (JavaThread *) thread;
	    ThreadBlockInVM tbivm(jt);
	
	    jt->set_suspend_equivalent();
	    // cleared by handle_special_suspend_equivalent_condition() or
	    // java_suspend_self() via check_and_wait_while_suspended()
	
	    HANDLE events[1];
	    events[0] = osthread->interrupt_event();
	    HighResolutionInterval *phri=NULL;
	    if(!ForceTimeHighResolution)
	      phri = new HighResolutionInterval( ms );
	    if (WaitForMultipleObjects(1, events, FALSE, (DWORD)ms) == WAIT_TIMEOUT) {
	      result = OS_TIMEOUT;
	    } else {
	      ResetEvent(osthread->interrupt_event());
	      osthread->set_interrupted(false);
	      result = OS_INTRPT;
	    }
	    delete phri; //if it is NULL, harmless
	
	    // were we externally suspended while we were waiting?
	    jt->check_and_wait_while_suspended();

  {- -------------------------------------------
  (1) (以下が, java.lang.Thread.interrupt() による割り込みを許さない場合の処理)
  
      Sleep() システムコールを呼んで, 指定時間分ねむるだけ.
      ---------------------------------------- -}

	  } else {
	    assert(!thread->is_Java_thread(), "must not be java thread");
	    Sleep((long) ms);
	    result = OS_TIMEOUT;
	  }

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return result;
	}
	
```


