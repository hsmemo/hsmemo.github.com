---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/management/ThreadInfo.java
### 説明(description)

```
    /**
     * Returns the total number of times that
     * the thread associated with this <tt>ThreadInfo</tt>
     * waited for notification.
     * I.e. the number of times that a thread has been
     * in the {@link java.lang.Thread.State#WAITING WAITING}
     * or {@link java.lang.Thread.State#TIMED_WAITING TIMED_WAITING} state.
     *
     * @return the total number of times that the thread
     * was in the <tt>WAITING</tt> or <tt>TIMED_WAITING</tt> state.
     */
```

### 名前(function name)
```
    public long getWaitedCount() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) waitedCount フィールドの値をリターンするだけ.
      ---------------------------------------- -}

	        return waitedCount;
	    }
	
```


