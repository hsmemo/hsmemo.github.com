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
     * Waits at most {@code millis} milliseconds for this thread to
     * die. A timeout of {@code 0} means to wait forever.
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
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
```

### 名前(function name)
```
    public final synchronized void join(long millis)
    throws InterruptedException {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) join 対象の Thread オブジェクトに対して java.lang.Object.wait() を呼び出し, 
      そのスレッドが終了するまで待つ. 
      (wait() が解けたら java.lang.Thread.isAlive() で生死判定を行い, 
       きちんと死亡が確認されるまで wait() を繰り返す)
      (なお, このメソッドは synchronized メソッドなので, 
       isAlive() による確認と wait() の呼び出しはアトミックに行われる)
  	
      (なお, 引数で timeup 時間が指定されている場合(引数の millis が 0 以外の値の場合)は, 
       最大でその時間だけ待つ)
      ---------------------------------------- -}

	        long base = System.currentTimeMillis();
	        long now = 0;
	
	        if (millis < 0) {
	            throw new IllegalArgumentException("timeout value is negative");
	        }
	
	        if (millis == 0) {
	            while (isAlive()) {
	                wait(0);
	            }
	        } else {
	            while (isAlive()) {
	                long delay = millis - now;
	                if (delay <= 0) {
	                    break;
	                }
	                wait(delay);
	                now = System.currentTimeMillis() - base;
	            }
	        }
	    }
	
```


