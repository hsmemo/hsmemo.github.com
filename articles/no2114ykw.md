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
     * Forces the thread to stop executing.
     * <p>
     * If there is a security manager installed, the <code>checkAccess</code>
     * method of this thread is called, which may result in a
     * <code>SecurityException</code> being raised (in the current thread).
     * <p>
     * If this thread is different from the current thread (that is, the current
     * thread is trying to stop a thread other than itself) or
     * <code>obj</code> is not an instance of <code>ThreadDeath</code>, the
     * security manager's <code>checkPermission</code> method (with the
     * <code>RuntimePermission("stopThread")</code> argument) is called in
     * addition.
     * Again, this may result in throwing a
     * <code>SecurityException</code> (in the current thread).
     * <p>
     * If the argument <code>obj</code> is null, a
     * <code>NullPointerException</code> is thrown (in the current thread).
     * <p>
     * The thread represented by this thread is forced to stop
     * whatever it is doing abnormally and to throw the
     * <code>Throwable</code> object <code>obj</code> as an exception. This
     * is an unusual action to take; normally, the <code>stop</code> method
     * that takes no arguments should be used.
     * <p>
     * It is permitted to stop a thread that has not yet been started.
     * If the thread is eventually started, it immediately terminates.
     *
     * @param      obj   the Throwable object to be thrown.
     * @exception  SecurityException  if the current thread cannot modify
     *               this thread.
     * @throws     NullPointerException if obj is <tt>null</tt>.
     * @see        #interrupt()
     * @see        #checkAccess()
     * @see        #run()
     * @see        #start()
     * @see        #stop()
     * @see        SecurityManager#checkAccess(Thread)
     * @see        SecurityManager#checkPermission
     * @deprecated This method is inherently unsafe.  See {@link #stop()}
     *        for details.  An additional danger of this
     *        method is that it may be used to generate exceptions that the
     *        target thread is unprepared to handle (including checked
     *        exceptions that the thread could not possibly throw, were it
     *        not for this method).
     *        For more information, see
     *        <a href="{@docRoot}/../technotes/guides/concurrency/threadPrimitiveDeprecation.html">Why
     *        are Thread.stop, Thread.suspend and Thread.resume Deprecated?</a>.
     */
```

### 名前(function name)
```
    @Deprecated
    public final synchronized void stop(Throwable obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし引数が null だったら NullPointerException.
      ---------------------------------------- -}

	        if (obj == null)
	            throw new NullPointerException();
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        SecurityManager security = System.getSecurityManager();
	        if (security != null) {
	            checkAccess();
	            if ((this != Thread.currentThread()) ||
	                (!(obj instanceof ThreadDeath))) {
	                security.checkPermission(SecurityConstants.STOP_THREAD_PERMISSION);
	            }
	        }

  {- -------------------------------------------
  (1) もしも suspend されている状態だと面倒なので, 念のために java.lang.Thread.resume() を呼び出しておく.
  
      (なお, まだ start() されていないスレッドの場合には, 必要が無いのでこの処理は省略する.
       start されていないスレッドでは, threadStatus フィールドは 0 になっている)
      ---------------------------------------- -}

	        // A zero status value corresponds to "NEW", it can't change to
	        // not-NEW because we hold the lock.
	        if (threadStatus != 0) {
	            resume(); // Wake up thread if it was suspended; no-op otherwise
	        }
	
  {- -------------------------------------------
  (1) java.lang.Thread.stop0() を呼び出してスレッドを停止させる.
      ---------------------------------------- -}

	        // The VM can handle all thread states
	        stop0(obj);
	    }
	
```


