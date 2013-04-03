---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.cpp

### 名前(function name)
```
static void do_resume(OSThread* osthread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(osthread->sr.is_suspended(), "thread should be suspended");

  {- -------------------------------------------
  (1) os::Linux::SuspendResume::set_suspend_action() を呼んで
      処理対象の OSThread の sr フィールドに SR_CONTINUE をセットした後, 
      そのスレッドに対して pthread_kill() で SR_signum を送信する.
      ---------------------------------------- -}

	  osthread->sr.set_suspend_action(SR_CONTINUE);
	
	  int status = pthread_kill(osthread->pthread_id(), SR_signum);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_status(status == 0, status, "pthread_kill");

  {- -------------------------------------------
  (1) 処理対象のスレッドが実際に起床するまでここで少し待機 (See: SR_handler()).
      ---------------------------------------- -}

	  // check status and wait unit notified of resumption
	  if (status == 0) {
	    for (int i = 0; osthread->sr.is_suspended(); i++) {
	      os::yield_all(i);
	    }
	  }

  {- -------------------------------------------
  (1) os::Linux::SuspendResume::set_suspend_action() を呼んで
      処理対象の OSThread の sr フィールドを SR_NONE に戻しておく.
      ---------------------------------------- -}

	  osthread->sr.set_suspend_action(SR_NONE);
	}
	
```


