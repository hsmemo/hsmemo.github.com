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
     * Notifies the group that the thread {@code t} has terminated.
     *
     * <p> Destroy the group if all of the following conditions are
     * true: this is a daemon thread group; there are no more alive
     * or unstarted threads in the group; there are no subgroups in
     * this thread group.
     *
     * @param  t
     *         the Thread that has terminated
     */
```

### 名前(function name)
```
    void threadTerminated(Thread t) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下の処理は, このインスタンスのロックを取った状態で行う)
      ---------------------------------------- -}

	        synchronized (this) {

  {- -------------------------------------------
  (1) java.lang.ThreadGroup.remove() で, 
      指定された Thread オブジェクトを threads フィールドから除去する 
      (同時に nthreads フィールドのデクリメントも行う).
      ---------------------------------------- -}

	            remove(t);
	
  {- -------------------------------------------
  (1) もし稼働中のスレッドがいなくなったら (= nthreads が 0 になったら), 
      この ThreadGroup 自身に対して java.lang.Object.notifyAll() を呼び出す.
      (これはどういう処理?? Thread.join() みたいな感じで終了を待ち受ける為に使う?? #TODO)
      ---------------------------------------- -}

	            if (nthreads == 0) {
	                notifyAll();
	            }

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	            if (daemon && (nthreads == 0) &&
	                (nUnstartedThreads == 0) && (ngroups == 0))
	            {
	                destroy();
	            }
	        }
	    }
	
```


