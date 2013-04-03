---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/os.cpp
### 説明(description)

```
// Serialize all thread state variables
```

### 名前(function name)
```
void os::serialize_thread_states() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) os::protect_memory() を呼んで, 
      serialize page の write 権限を落とし (= read-only にし), 直後に復活させる.
    
      (これが "pseudo remote membar" 処理.
       See: SafepointSynchronize::begin(), 
            InterpreterGenerator::generate_native_entry(), SharedRuntime::generate_native_wrapper())
  
  
      (なお, この処理全体は SerializePageLock で排他した状態で行っている.
       このロックは, os::block_on_serialize_page_trap() との間で排他を取るために使われている.
  
       ロックを取っている理由は, 
       read-only にした後で read-write に戻す前にこのスレッドが descheduling されると,
       JavaThread 側で, メモリアクセス違反 -> シグナルハンドラ -> 再度やり直し -> メモリアクセス違反 -> ...
       という処理が連続して長い停止時間が発生することがあったから, らしい (bug 6546278 も参照).
       
       See: os::block_on_serialize_page_trap())
      ---------------------------------------- -}

	  // On some platforms such as Solaris & Linux, the time duration of the page
	  // permission restoration is observed to be much longer than expected  due to
	  // scheduler starvation problem etc. To avoid the long synchronization
	  // time and expensive page trap spinning, 'SerializePageLock' is used to block
	  // the mutator thread if such case is encountered. See bug 6546278 for details.
	  Thread::muxAcquire(&SerializePageLock, "serialize_thread_states");
	  os::protect_memory((char *)os::get_memory_serialize_page(),
	                     os::vm_page_size(), MEM_PROT_READ);
	  os::protect_memory((char *)os::get_memory_serialize_page(),
	                     os::vm_page_size(), MEM_PROT_RW);
	  Thread::muxRelease(&SerializePageLock);
	}
	
```


