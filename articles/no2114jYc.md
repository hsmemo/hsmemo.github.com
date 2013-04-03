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
     * Causes this thread to begin execution; the Java Virtual Machine
     * calls the <code>run</code> method of this thread.
     * <p>
     * The result is that two threads are running concurrently: the
     * current thread (which returns from the call to the
     * <code>start</code> method) and the other thread (which executes its
     * <code>run</code> method).
     * <p>
     * It is never legal to start a thread more than once.
     * In particular, a thread may not be restarted once it has completed
     * execution.
     *
     * @exception  IllegalThreadStateException  if the thread was already
     *               started.
     * @see        #run()
     * @see        #stop()
     */
```

### 名前(function name)
```
    public synchronized void start() {
```

### 本体部(body)
```
	        /**
	         * This method is not invoked for the main method thread or "system"
	         * group threads created/set up by the VM. Any new functionality added
	         * to this method in the future may have to also be added to the VM.
	         *
	         * A zero status value corresponds to state "NEW".
	         */

  {- -------------------------------------------
  (1) (生成直後のスレッドでは, threadStatus フィールドは 0 になっているはず)
      既に start 済みのスレッドにもう一度 start() が呼ばれた場合, IllegalThreadStateException.
      ---------------------------------------- -}

	        if (threadStatus != 0)
	            throw new IllegalThreadStateException();
	
  {- -------------------------------------------
  (1) java.lang.ThreadGroup.add() で, ThreadGroup にこのスレッドを追加する.
      ---------------------------------------- -}

	        /* Notify the group that this thread is about to be started
	         * so that it can be added to the group's list of threads
	         * and the group's unstarted count can be decremented. */
	        group.add(this);
	
  {- -------------------------------------------
  (1) java.lang.ThreadGroup.start0() で, このスレッドを開始させる.
  
      (なお, もし java.lang.ThreadGroup.start0() が失敗した(例外が出た)場合には, 
      java.lang.ThreadGroup.threadStartFailed() で ThreadGroup からこのスレッドを除去している)
      ---------------------------------------- -}

	        boolean started = false;
	        try {
	            start0();
	            started = true;
	        } finally {
	            try {
	                if (!started) {
	                    group.threadStartFailed(this);
	                }
	            } catch (Throwable ignore) {
	                /* do nothing. If start0 threw a Throwable then
	                  it will be passed up the call stack */
	            }
	        }
	    }
	
```


