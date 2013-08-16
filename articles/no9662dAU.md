---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp
### 説明(description)

```
/*
 * The Windows implementation of Park is very straightforward: Basic
 * operations on Win32 Events turn out to have the right semantics to
 * use them directly. We opportunistically resuse the event inherited
 * from Monitor.
 */


```

### 名前(function name)
```
void Parker::park(bool isAbsolute, jlong time) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee (_ParkEvent != NULL, "invariant") ;

  {- -------------------------------------------
  (1) 初めに, 指定された待ち時間の値(time)を調整しておく.
      * 指定が絶対時間で, かつ値が 0 の場合
        INFINITE とする
      * 指定が絶対時間で, 値が 0 以外の場合
        現在時刻からの相対時間に修正する
      * 指定が相対時間の場合
        1000000 で割る (ナノ秒からミリ秒へと変換する)
    
      ただし, 以下の場合はここでリターンする.
      * 指定された待ち時間が負値の場合
      * 指定された待ち時間が絶対時間であり, その時間が現在時刻よりも古い場合
      ---------------------------------------- -}

	  // First, demultiplex/decode time arguments
	  if (time < 0) { // don't wait
	    return;
	  }
	  else if (time == 0 && !isAbsolute) {
	    time = INFINITE;
	  }
	  else if  (isAbsolute) {
	    time -= os::javaTimeMillis(); // convert to relative time
	    if (time <= 0) // already elapsed
	      return;
	  }
	  else { // relative
	    time /= 1000000; // Must coarsen from nanos to millis
	    if (time == 0)   // Wait for the minimal time unit if zero
	      time = 1;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread* thread = (JavaThread*)(Thread::current());
	  assert(thread->is_Java_thread(), "Must be JavaThread");
	  JavaThread *jt = (JavaThread *)thread;
	
  {- -------------------------------------------
  (1) 以下の場合には, ここでリターン.
      * 処理対象のスレッドに対して java.lang.Thread.interrupt() が呼ばれていた場合
      * 既にシグナルを送られていた場合 (= 待ち時間 0 の WaitForSingleObject() が WAIT_OBJECT_0 を返した場合)
      ---------------------------------------- -}

	  // Don't wait if interrupted or already triggered
	  if (Thread::is_interrupted(thread, false) ||
	    WaitForSingleObject(_ParkEvent, 0) == WAIT_OBJECT_0) {
	    ResetEvent(_ParkEvent);
	    return;
	  }

  {- -------------------------------------------
  (1) ThreadBlockInVM 及び OSThreadWaitState で JavaThread/OSThread の状態を変更した後, 
      WaitForSingleObject() を呼んで誰かが unpark() してくれるまで眠りにつく.
      目が覚めたら, ResetEvent() でイベントオブジェクトを非シグナル状態に戻す.
  
      (なお, 寝ている間に java.lang.Thread.suspend() で suspend 状態にされているかもしれないので, 
       目が覚めた後に JavaThread::handle_special_suspend_equivalent_condition() によるチェックも行っている.
       もし suspend されていれば, JavaThread::java_suspend_self() でサスペンドが解除されるまで眠りにつく.)
  
  
      #TODO  JavaThread::set_suspend_equivalent() はどういう意味がある??
      ---------------------------------------- -}

	  else {
	    ThreadBlockInVM tbivm(jt);
	    OSThreadWaitState osts(thread->osthread(), false /* not Object.wait() */);
	    jt->set_suspend_equivalent();
	
	    WaitForSingleObject(_ParkEvent,  time);
	    ResetEvent(_ParkEvent);
	
	    // If externally suspended while waiting, re-suspend
	    if (jt->handle_special_suspend_equivalent_condition()) {
	      jt->java_suspend_self();
	    }
	  }
	}
	
```


