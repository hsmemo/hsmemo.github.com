---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/Shutdown.java
### 説明(description)

```
    /* The actual shutdown sequence is defined here.
     *
     * If it weren't for runFinalizersOnExit, this would be simple -- we'd just
     * run the hooks and then halt.  Instead we need to keep track of whether
     * we're running hooks or finalizers.  In the latter case a finalizer could
     * invoke exit(1) to cause immediate termination, while in the former case
     * any further invocations of exit(n), for any n, simply stall.  Note that
     * if on-exit finalizers are enabled they're run iff the shutdown is
     * initiated by an exit(0); they're never run on exit(n) for n != 0 or in
     * response to SIGINT, SIGTERM, etc.
     */
```

### 名前(function name)
```
    private static void sequence() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        synchronized (lock) {
	            /* Guard against the possibility of a daemon thread invoking exit
	             * after DestroyJavaVM initiates the shutdown sequence
	             */
	            if (state != HOOKS) return;
	        }

  {- -------------------------------------------
  (1) java.lang.Shutdown.runHooks() を呼んで, shutdown hook を実行する.
      ---------------------------------------- -}

	        runHooks();

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	        boolean rfoe;

  {- -------------------------------------------
  (1) state を変更しておく
      ---------------------------------------- -}

	        synchronized (lock) {
	            state = FINALIZERS;
	            rfoe = runFinalizersOnExit;
	        }

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        if (rfoe) runAllFinalizers();
	
```


