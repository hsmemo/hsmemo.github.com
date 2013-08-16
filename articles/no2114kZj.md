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
     * Waits at most {@code millis} milliseconds plus
     * {@code nanos} nanoseconds for this thread to die.
     *
     * <p> This implementation uses a loop of {@code this.wait} calls
     * conditioned on {@code this.isAlive}. As a thread terminates the
     * {@code this.notifyAll} method is invoked. It is recommended that
     * applications not use {@code wait}, {@code notify}, or
     * {@code notifyAll} on {@code Thread} instances.
     *
     * @param  millis
     *         the time to wait in milliseconds
     *
     * @param  nanos
     *         {@code 0-999999} additional nanoseconds to wait
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative, or the value
     *          of {@code nanos} is not in the range {@code 0-999999}
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
```

### 名前(function name)
```
    public final synchronized void join(long millis, int nanos)
    throws InterruptedException {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし引数が不正であれば IllegalArgumentException.
      ---------------------------------------- -}
	
	        if (millis < 0) {
	            throw new IllegalArgumentException("timeout value is negative");
	        }
	
	        if (nanos < 0 || nanos > 999999) {
	            throw new IllegalArgumentException(
	                                "nanosecond timeout value out of range");
	        }
	
  {- -------------------------------------------
  (1) 以下のどちらかが成り立つ場合は millis 引数を 1 インクリメントしておく
  
      * nanos 引数が 500000 以上の値 (= つまり 500 usec 以上)
      * millis 引数が 0, かつ nanos 引数が 1 以上の値
      ---------------------------------------- -}

	        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
	            millis++;
	        }
	
  {- -------------------------------------------
  (1) java.lang.Thread.join() を呼んで, 対象のスレッドの終了待ちを行う.
      ---------------------------------------- -}

	        join(millis);
	    }
	
```


