---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp

### 名前(function name)
```
OSReturn os::set_native_priority(Thread* thread, int newpri) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(newpri >= MinimumPriority && newpri <= MaximumPriority, "bad priority mapping");

  {- -------------------------------------------
  (1) thr_setprio() システムコールで優先度を変更する.
      さらに, 以下のどちらかが成り立つ場合は, set_lwp_priority() で LWP レベルの優先度も変更する.
      * T2 libthread を使っている場合
      * UseBoundThreads オプションが指定されており, 対象のスレッドが HotSpot 内で作られたものである場合
      
      (ただし, UseThreadPriorities オプションが指定されていない場合には, 何もしない)
      ---------------------------------------- -}

	  if ( !UseThreadPriorities ) return OS_OK;
	  int status = thr_setprio(thread->osthread()->thread_id(), newpri);
	  if ( os::Solaris::T2_libthread() || (UseBoundThreads && thread->osthread()->is_vm_created()) )
	    status |= (set_lwp_priority (thread->osthread()->thread_id(),
	                    thread->osthread()->lwp_id(), newpri ));
	  return (status == 0) ? OS_OK : OS_ERR;
	}
	
```


