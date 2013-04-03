---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.cpp
### 説明(description)

```
//
// Handler function invoked when a thread's execution is suspended or
// resumed. We have to be careful that only async-safe functions are
// called here (Note: most pthread functions are not async safe and
// should be avoided.)
//
// Note: sigwait() is a more natural fit than sigsuspend() from an
// interface point of view, but sigwait() prevents the signal hander
// from being run. libpthread would get very confused by not having
// its signal handlers run and prevents sigwait()'s use with the
// mutex granting granting signal.
//
// Currently only ever called on the VMThread
//
```

### 名前(function name)
```
static void SR_handler(int sig, siginfo_t* siginfo, ucontext_t* context) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Save and restore errno to avoid confusing native code with EINTR
	  // after sigsuspend.
	  int old_errno = errno;
	
	  Thread* thread = Thread::current();
	  OSThread* osthread = thread->osthread();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(thread->is_VM_thread(), "Must be VMThread");

  {- -------------------------------------------
  (1) 以下の処理は, サスペンドされた場合.
      
      サスペンドされた場合 (= sr フィールドが SR_SUSPEND の場合) は, 
      os::Linux::SuspendResume::set_suspended() を呼んで sr フィールドの SR_SUSPENDED ビットを立てた後, 
      sigsuspend() を呼んで眠りにつく.
      (sr フィールドが SR_CONTINUE に変更されてから SR_signum が再度送信されるまで, ここで待機)
    
      (なお, 眠りから覚めたら resume_clear_context() を呼んで sr フィールドをリセットしている)
      ---------------------------------------- -}

	  // read current suspend action
	  int action = osthread->sr.suspend_action();
	  if (action == SR_SUSPEND) {
	    suspend_save_context(osthread, siginfo, context);
	
	    // Notify the suspend action is about to be completed. do_suspend()
	    // waits until SR_SUSPENDED is set and then returns. We will wait
	    // here for a resume signal and that completes the suspend-other
	    // action. do_suspend/do_resume is always called as a pair from
	    // the same thread - so there are no races
	
	    // notify the caller
	    osthread->sr.set_suspended();
	
	    sigset_t suspend_set;  // signals for sigsuspend()
	
	    // get current set of blocked signals and unblock resume signal
	    pthread_sigmask(SIG_BLOCK, NULL, &suspend_set);
	    sigdelset(&suspend_set, SR_signum);
	
	    // wait here until we are resumed
	    do {
	      sigsuspend(&suspend_set);
	      // ignore all returns until we get a resume signal
	    } while (osthread->sr.suspend_action() != SR_CONTINUE);
	
	    resume_clear_context(osthread);
	
  {- -------------------------------------------
  (1) 以下の処理は, レジュームされた場合.
      ---------------------------------------- -}

	  } else {
	    assert(action == SR_CONTINUE, "unexpected sr action");
	    // nothing special to do - just leave the handler
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  errno = old_errno;
	}
	
```


