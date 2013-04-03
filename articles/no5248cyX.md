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
static int SR_initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  struct sigaction act;
	  char *s;

  {- -------------------------------------------
  (1) _JAVA_SR_SIGNUM 環境変数が指定されていれば, 
      SR_signum(= suspend/resume に使用するシグナル番号) を指定の値に変更しておく.
      ---------------------------------------- -}

	  /* Get signal number to use for suspend/resume */
	  if ((s = ::getenv("_JAVA_SR_SIGNUM")) != 0) {
	    int sig = ::strtol(s, 0, 10);
	    if (sig > 0 || sig < _NSIG) {
	        SR_signum = sig;
	    }
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SR_signum > SIGSEGV && SR_signum > SIGBUS,
	        "SR_signum must be greater than max(SIGSEGV, SIGBUS), see 4355769");
	
  {- -------------------------------------------
  (1) sigaction() を呼んで, SR_signum 用のシグナルハンドラとして SR_handler() を登録する.
      ---------------------------------------- -}

	  sigemptyset(&SR_sigset);
	  sigaddset(&SR_sigset, SR_signum);
	
	  /* Set up signal handler for suspend/resume */
	  act.sa_flags = SA_RESTART|SA_SIGINFO;
	  act.sa_handler = (void (*)(int)) SR_handler;
	
	  // SR_signum is blocked by default.
	  // 4528190 - We also need to block pthread restart signal (32 on all
	  // supported Linux platforms). Note that LinuxThreads need to block
	  // this signal for all threads to work properly. So we don't have
	  // to use hard-coded signal number when setting up the mask.
	  pthread_sigmask(SIG_BLOCK, NULL, &act.sa_mask);
	
	  if (sigaction(SR_signum, &act, 0) == -1) {
	    return -1;
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Save signal flag
	  os::Linux::set_our_sigflags(SR_signum, act.sa_flags);
	  return 0;
	}
	
```


