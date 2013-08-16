---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/Thread.java
### 説明(description)

```
    /**
     * Interrupts this thread.
     *
     * <p> Unless the current thread is interrupting itself, which is
     * always permitted, the {@link #checkAccess() checkAccess} method
     * of this thread is invoked, which may cause a {@link
     * SecurityException} to be thrown.
     *
     * <p> If this thread is blocked in an invocation of the {@link
     * Object#wait() wait()}, {@link Object#wait(long) wait(long)}, or {@link
     * Object#wait(long, int) wait(long, int)} methods of the {@link Object}
     * class, or of the {@link #join()}, {@link #join(long)}, {@link
     * #join(long, int)}, {@link #sleep(long)}, or {@link #sleep(long, int)},
     * methods of this class, then its interrupt status will be cleared and it
     * will receive an {@link InterruptedException}.
     *
     * <p> If this thread is blocked in an I/O operation upon an {@link
     * java.nio.channels.InterruptibleChannel </code>interruptible
     * channel<code>} then the channel will be closed, the thread's interrupt
     * status will be set, and the thread will receive a {@link
     * java.nio.channels.ClosedByInterruptException}.
     *
     * <p> If this thread is blocked in a {@link java.nio.channels.Selector}
     * then the thread's interrupt status will be set and it will return
     * immediately from the selection operation, possibly with a non-zero
     * value, just as if the selector's {@link
     * java.nio.channels.Selector#wakeup wakeup} method were invoked.
     *
     * <p> If none of the previous conditions hold then this thread's interrupt
     * status will be set. </p>
     *
     * <p> Interrupting a thread that is not alive need not have any effect.
     *
     * @throws  SecurityException
     *          if the current thread cannot modify this thread
     *
     * @revised 6.0
     * @spec JSR-51
     */
```

### 名前(function name)
```
    public void interrupt() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        if (this != Thread.currentThread())
	            checkAccess();
	
  {- -------------------------------------------
  (1) java.nio のメソッド内でブロックしている場合(= blocker フィールドが null ではない場合)は, 
      java.lang.Thread.interrupt0() を呼び出して interrupt フラグを立てた後, 
      blocker フィールドのオブジェクトに対して sun.nio.ch.Interruptible.interrupt() を呼び出して処理を行う.
      ---------------------------------------- -}

	        synchronized (blockerLock) {
	            Interruptible b = blocker;
	            if (b != null) {
	                interrupt0();           // Just to set the interrupt flag
	                b.interrupt(this);
	                return;
	            }
	        }

  {- -------------------------------------------
  (1) そうでなければ, java.lang.Thread.interrupt0() で処理を行う.
      ---------------------------------------- -}

	        interrupt0();
	    }
	
```


