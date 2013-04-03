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
     * Increments the count of unstarted threads in the thread group.
     * Unstarted threads are not added to the thread group so that they
     * can be collected if they are never started, but they must be
     * counted so that daemon thread groups with unstarted threads in
     * them are not destroyed.
     */
```

### 名前(function name)
```
    void addUnstarted() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) nUnstartedThreads フィールドの値をインクリメントするだけ.
      (ただし, この ThreadGroup が既に破棄されている場合は, IllegalThreadStateException を出す)
      ---------------------------------------- -}

	        synchronized(this) {
	            if (destroyed) {
	                throw new IllegalThreadStateException();
	            }
	            nUnstartedThreads++;
	        }
	    }
	
```


