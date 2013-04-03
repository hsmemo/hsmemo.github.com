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
     * Returns the approximate accumulated elapsed time (in milliseconds)
     * that the thread associated with this <tt>ThreadInfo</tt>
     * has blocked to enter or reenter a monitor
     * since thread contention monitoring is enabled.
     * I.e. the total accumulated time the thread has been in the
     * {@link java.lang.Thread.State#BLOCKED BLOCKED} state since thread
     * contention monitoring was last enabled.
     * This method returns <tt>-1</tt> if thread contention monitoring
     * is disabled.
     *
     * <p>The Java virtual machine may measure the time with a high
     * resolution timer.  This statistic is reset when
     * the thread contention monitoring is reenabled.
     *
     * @return the approximate accumulated elapsed time in milliseconds
     * that a thread entered the <tt>BLOCKED</tt> state;
     * <tt>-1</tt> if thread contention monitoring is disabled.
     *
     * @throws java.lang.UnsupportedOperationException if the Java
     * virtual machine does not support this operation.
     *
     * @see ThreadMXBean#isThreadContentionMonitoringSupported
     * @see ThreadMXBean#setThreadContentionMonitoringEnabled
     */
```

### 名前(function name)
```
    public long getBlockedTime() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) blockedTime フィールドの値をリターンするだけ.
      ---------------------------------------- -}

	        return blockedTime;
	    }
	
```


