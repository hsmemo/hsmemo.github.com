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
     * Returns a map of stack traces for all live threads.
     * The map keys are threads and each map value is an array of
     * <tt>StackTraceElement</tt> that represents the stack dump
     * of the corresponding <tt>Thread</tt>.
     * The returned stack traces are in the format specified for
     * the {@link #getStackTrace getStackTrace} method.
     *
     * <p>The threads may be executing while this method is called.
     * The stack trace of each thread only represents a snapshot and
     * each stack trace may be obtained at different time.  A zero-length
     * array will be returned in the map value if the virtual machine has
     * no stack trace information about a thread.
     *
     * <p>If there is a security manager, then the security manager's
     * <tt>checkPermission</tt> method is called with a
     * <tt>RuntimePermission("getStackTrace")</tt> permission as well as
     * <tt>RuntimePermission("modifyThreadGroup")</tt> permission
     * to see if it is ok to get the stack trace of all threads.
     *
     * @return a <tt>Map</tt> from <tt>Thread</tt> to an array of
     * <tt>StackTraceElement</tt> that represents the stack trace of
     * the corresponding thread.
     *
     * @throws SecurityException
     *        if a security manager exists and its
     *        <tt>checkPermission</tt> method doesn't allow
     *        getting the stack trace of thread.
     * @see #getStackTrace
     * @see SecurityManager#checkPermission
     * @see RuntimePermission
     * @see Throwable#getStackTrace
     *
     * @since 1.5
     */
```

### 名前(function name)
```
    public static Map<Thread, StackTraceElement[]> getAllStackTraces() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし SecurityManager が登録されていれば, チェックを行う.
      ---------------------------------------- -}

	        // check for getStackTrace permission
	        SecurityManager security = System.getSecurityManager();
	        if (security != null) {
	            security.checkPermission(
	                SecurityConstants.GET_STACK_TRACE_PERMISSION);
	            security.checkPermission(
	                SecurityConstants.MODIFY_THREADGROUP_PERMISSION);
	        }
	
  {- -------------------------------------------
  (1) java.lang.Thread.getThreads() で全スレッドの一覧を取得し, 
      それを引数として java.lang.Thread.dumpThreads() を呼び出してスタックトレースを取得.
      
      (なお, 結果は HashMap として返す必要があるので, 
       java.lang.Thread.dumpThreads() の呼び出しの後で
       結果を配列から HashMap に詰め直してリターンする)
      ---------------------------------------- -}

	        // Get a snapshot of the list of all threads
	        Thread[] threads = getThreads();
	        StackTraceElement[][] traces = dumpThreads(threads);
	        Map<Thread, StackTraceElement[]> m = new HashMap<>(threads.length);
	        for (int i = 0; i < threads.length; i++) {
	            StackTraceElement[] stackTrace = traces[i];
	            if (stackTrace != null) {
	                m.put(threads[i], stackTrace);
	            }
	            // else terminated so we don't put it in the map
	        }
	        return m;
	    }
	
```


