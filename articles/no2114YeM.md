---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp
### 説明(description)
(コメントで書かれている警告によると, 
 Solaris の os::yield() はスレッドの状態遷移(thread-state transition)を起こすが, 
 Linux や Windows 版の実装では起こさないので, これについてはチェックすべき, とのこと.)

```
// Caveat: Solaris os::yield() causes a thread-state transition whereas
// the linux and win32 implementations do not.  This should be checked.

```

### 名前(function name)
```
void os::yield() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) os::sleep() を呼び出すだけ.
      ---------------------------------------- -}

	  // Yields to all threads with same or greater priority
	  os::sleep(Thread::current(), 0, false);
	}
	
```


