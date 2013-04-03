---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/ThreadGroup.java
### 説明(description)

```
    /**
     * Adds the specified thread to this thread group.
     *
     * <p> Note: This method is called from both library code
     * and the Virtual Machine. It is called from VM to add
     * certain system threads to the system thread group.
     *
     * @param  t
     *         the Thread to be added
     *
     * @throws  IllegalThreadStateException
     *          if the Thread group has been destroyed
     */
```

### 名前(function name)
```
    void add(Thread t) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数で指定された Thread オブジェクトを, threads フィールドに格納する.
  
      (ついでに, nthreads フィールドの値をインクリメントと nUnstartedThreads フィールドの値をデクリメントも行っている)
      (ただし, この ThreadGroup が既に破棄されている場合は, IllegalThreadStateException を出す)
      (また, threads フィールドが未初期化もしくは一杯の場合には, 新しい配列の確保も行っている)
      ---------------------------------------- -}

	        synchronized (this) {
	            if (destroyed) {
	                throw new IllegalThreadStateException();
	            }
	            if (threads == null) {
	                threads = new Thread[4];
	            } else if (nthreads == threads.length) {
	                threads = Arrays.copyOf(threads, nthreads * 2);
	            }
	            threads[nthreads] = t;
	
	            // This is done last so it doesn't matter in case the
	            // thread is killed
	            nthreads++;
	
	            // The thread is now a fully fledged member of the group, even
	            // though it may, or may not, have been started yet. It will prevent
	            // the group from being destroyed so the unstarted Threads count is
	            // decremented.
	            nUnstartedThreads--;
	        }
	    }
	
```


