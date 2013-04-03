---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/fprofiler.cpp

### 名前(function name)
```
void FlatProfiler::record_vm_tick() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ProfileVM オプションが指定されていない場合は, 何もしない)
      ---------------------------------------- -}

	  // Profile the VM Thread itself if needed
	  // This is done without getting the Threads_lock and we can go deep
	  // inside Safepoint, etc.
	  if( ProfileVM  ) {

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    ResourceMark rm;
	    ExtendedPC epc;
	    const char *name = NULL;
	    char buf[256];
	    buf[0] = '\0';
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    vm_thread_profiler->inc_thread_ticks();
	
  {- -------------------------------------------
  (1) os::get_thread_pc() を呼んで現在の VMThread の pc を取得し, 
      その pc 情報を基に os::dll_address_to_function_name() で実行している関数名を取得する.
    
      関数名が取得できたら, ThreadProfiler::vm_update() を呼んでプロファイル情報を更新しておく.
      (pc や関数名が取得できなかった場合は何もしない)
      ---------------------------------------- -}

	    // Get a snapshot of a current VMThread pc (and leave it running!)
	    // The call may fail if, for instance the VM thread is interrupted while
	    // holding the Interrupt_lock or for other reasons.
	    epc = os::get_thread_pc(VMThread::vm_thread());
	    if(epc.pc() != NULL) {
	      if (os::dll_address_to_function_name(epc.pc(), buf, sizeof(buf), NULL)) {
	         name = buf;
	      }
	    }
	    if (name != NULL) {
	      vm_thread_profiler->vm_update(name, tp_native);
	    }
	  }
	}
	
```


