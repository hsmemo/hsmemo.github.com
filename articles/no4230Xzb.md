---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/java.cpp

### 名前(function name)
```
void vm_exit(int code) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし ThreadLocalStorage からカレントスレッドが取得できない場合
      (<= これは何か致命的な問題が生じている場合), 
      vm_direct_exit() ですぐに終了.
      ---------------------------------------- -}

	  Thread* thread = ThreadLocalStorage::is_initialized() ?
	    ThreadLocalStorage::get_thread_slow() : NULL;
	  if (thread == NULL) {
	    // we have serious problems -- just exit
	    vm_direct_exit(code);
	  }
	
  {- -------------------------------------------
  (1) まだ VMThread が生きていれば VM_Exit を実行する.
      (通常はここで終了.
       もし VMThread が何か失敗すれば, その後に vm_direct_exit() を実行して終了することになるらしい)
      ---------------------------------------- -}

	  if (VMThread::vm_thread() != NULL) {
	    // Fire off a VM_Exit operation to bring VM to a safepoint and exit
	    VM_Exit op(code);
	    if (thread->is_Java_thread())
	      ((JavaThread*)thread)->set_thread_state(_thread_in_vm);
	    VMThread::execute(&op);
	    // should never reach here; but in case something wrong with VM Thread.
	    vm_direct_exit(code);

  {- -------------------------------------------
  (1) VMThread が生きていなければ vm_direct_exit() を実行するだけ.
      ---------------------------------------- -}

	  } else {
	    // VM thread is gone, just exit
	    vm_direct_exit(code);
	  }
	  ShouldNotReachHere();
	
```


