---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp
### 説明(description)

```
// Allocate and initialize a new OSThread
```

### 名前(function name)
```
bool os::create_thread(Thread* thread, ThreadType thr_type, size_t stack_size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  unsigned thread_id;
	
  {- -------------------------------------------
  (1) OSThread オブジェクトを生成する (失敗したらここでリターン)
      ---------------------------------------- -}

	  // Allocate the OSThread object
	  OSThread* osthread = new OSThread(NULL, NULL);
	  if (osthread == NULL) {
	    return false;
	  }
	
  {- -------------------------------------------
  (1) CreateEvent() を呼んで, interrupt 処理で使用するイベントオブジェクトを生成する.
      出来たイベントオブジェクトは interrupt_event フィールドにセットしておく.
  
      (もし CreateEvent() が失敗したら, 
       生成した OSThread オブジェクトを開放した後, ここでリターン)
    
      (なお, このイベントオブジェクトは 
       java.lang.Thread.sleep() で寝ているスレッドを
       java.lang.Thread.interrupt() で強制的に起こす処理で使用される.
       See: os::sleep(), os::interrupt())
      ---------------------------------------- -}

	  // Initialize support for Java interrupts
	  HANDLE interrupt_event = CreateEvent(NULL, true, false, NULL);
	  if (interrupt_event == NULL) {
	    delete osthread;
	    return NULL;
	  }
	  osthread->set_interrupt_event(interrupt_event);
	  osthread->set_interrupted(false);
	
  {- -------------------------------------------
  (1) Thread::set_osthread() を呼んで, 生成した OSThread を
      引数で渡された Thread オブジェクトに対応づける
      ---------------------------------------- -}

	  thread->set_osthread(osthread);
	
  {- -------------------------------------------
  (1) 生成するスレッドのスタックサイズを決定する.
      決定方法は以下の通り.
  
      * 引数でスタックサイズが指定されている場合 (引数の stack_size が 0 ではない場合)
        その値に設定する
      * 引数でスタックサイズが指定されていない場合 (引数の stack_size が 0 の場合)
        スレッド種別に応じた以下のデフォルト値に設定する
        * JavaThread の場合
          JavaThread::stack_size_at_create() の値を使用
          (これは -Xss オプションで指定した値.
           See: Arguments::parse_each_vm_init_arg(), os::init_2())
        * CompilerThread の場合
          CompilerThreadStackSize オプションの値を使用.
          (ただし, CompilerThreadStackSize オプションが指定されていない場合(値が 0 の場合)には, 
           VMThread 等と同じ方法で決める)
        * VMThread, Parallel GC thread, Concurrent GC thread, WatcherThread の場合
          VMThreadStackSize オプションの値を使用.
          (ただし, VMThreadStackSize オプションが指定されていない場合(値が 0 の場合)には, 
           os::Linux::default_stack_size() の値を使用する)
      ---------------------------------------- -}

	  if (stack_size == 0) {
	    switch (thr_type) {
	    case os::java_thread:
	      // Java threads use ThreadStackSize which default value can be changed with the flag -Xss
	      if (JavaThread::stack_size_at_create() > 0)
	        stack_size = JavaThread::stack_size_at_create();
	      break;
	    case os::compiler_thread:
	      if (CompilerThreadStackSize > 0) {
	        stack_size = (size_t)(CompilerThreadStackSize * K);
	        break;
	      } // else fall through:
	        // use VMThreadStackSize if CompilerThreadStackSize is not defined
	    case os::vm_thread:
	    case os::pgc_thread:
	    case os::cgc_thread:
	    case os::watcher_thread:
	      if (VMThreadStackSize > 0) stack_size = (size_t)(VMThreadStackSize * K);
	      break;
	    }
	  }
	
  {- -------------------------------------------
  (1) (以下で, Win32 スレッドの生成処理を行う.
       なおコメントによると, スタックサイズについては以下のように扱っているとのこと.
  
       「MSDN のドキュメントに書かれている内容に反して, 
         _beginthreadex() で指定する "stack_size" はスタックサイズを指定するものではない.
         代わりに, 指定した値は初期状態で commmit されるスタック量を決定する.
         スタックサイズは実行可能ファイルの PE ヘッダー内の値で決まる.
       
         もし "stack_size" が PE ヘッダーのデフォルト量よりも大きい場合, 
         その大きさは 1MB 単位で切り上げられる.
       
         例えば launcher のデフォルトスタックサイズが 320K の場合, 
         320K 未満の stack_size を指定してもスタックサイズ自体には影響しない.
         代わりに stack_size は初期状態でのコミット量にのみ影響する.
  
         逆に, stack_size をデフォルトサイズより大きく指定した場合, メモリ使用量が増大する恐れがある.
         この理由は, 大きさが 1MB 単位で切り上げられるためだけでなく, 
         指定した量が全てコミットされるためでもある.
       
         なお, Windows XP で STACK_SIZE_PARAM_IS_A_RESERVATION というフラグが追加された.
         これは CreateThread() 用のフラグで, 
         指定すると "stack_size" をスタックサイズとして扱わせる効果がある.
         残念ながら HotSpot 内では C runtime library を使用するので, 
         CreateThread() は使えない (使えない理由は MSDN 参照).
         しかし, 嬉しいことに, このフラグは _beginthreadex() にも効くようだ.」)
      ---------------------------------------- -}

	  // Create the Win32 thread
	  //
	  // Contrary to what MSDN document says, "stack_size" in _beginthreadex()
	  // does not specify stack size. Instead, it specifies the size of
	  // initially committed space. The stack size is determined by
	  // PE header in the executable. If the committed "stack_size" is larger
	  // than default value in the PE header, the stack is rounded up to the
	  // nearest multiple of 1MB. For example if the launcher has default
	  // stack size of 320k, specifying any size less than 320k does not
	  // affect the actual stack size at all, it only affects the initial
	  // commitment. On the other hand, specifying 'stack_size' larger than
	  // default value may cause significant increase in memory usage, because
	  // not only the stack space will be rounded up to MB, but also the
	  // entire space is committed upfront.
	  //
	  // Finally Windows XP added a new flag 'STACK_SIZE_PARAM_IS_A_RESERVATION'
	  // for CreateThread() that can treat 'stack_size' as stack size. However we
	  // are not supposed to call CreateThread() directly according to MSDN
	  // document because JVM uses C runtime library. The good news is that the
	  // flag appears to work with _beginthredex() as well.
	
  {- -------------------------------------------
  (1) (STACK_SIZE_PARAM_IS_A_RESERVATION が定義されていない環境で
       コンパイルする場合のために, 一応ここで定義をしておく)
      ---------------------------------------- -}

	#ifndef STACK_SIZE_PARAM_IS_A_RESERVATION
	#define STACK_SIZE_PARAM_IS_A_RESERVATION  (0x10000)
	#endif
	
  {- -------------------------------------------
  (1) _beginthredex() でスレッドを生成する.
      (なお, スレッドのエントリポイントは java_start() としている)
      (また, CREATE_SUSPENDED で生成しているため, 
       実際に動き始めるのは os::pd_start_thread() で ResumeThread() されてから.
       See: os::pd_start_thread())
      ---------------------------------------- -}

	  HANDLE thread_handle =
	    (HANDLE)_beginthreadex(NULL,
	                           (unsigned)stack_size,
	                           (unsigned (__stdcall *)(void*)) java_start,
	                           thread,
	                           CREATE_SUSPENDED | STACK_SIZE_PARAM_IS_A_RESERVATION,
	                           &thread_id);

  {- -------------------------------------------
  (1) もしスレッド生成に失敗していた場合は, 
      STACK_SIZE_PARAM_IS_A_RESERVATION がサポートされていないだけかもしれないので, 
      もう一度 (STACK_SIZE_PARAM_IS_A_RESERVATION を付けずに) _beginthredex() を呼び出す.
      ---------------------------------------- -}

	  if (thread_handle == NULL) {
	    // perhaps STACK_SIZE_PARAM_IS_A_RESERVATION is not supported, try again
	    // without the flag.
	    thread_handle =
	    (HANDLE)_beginthreadex(NULL,
	                           (unsigned)stack_size,
	                           (unsigned (__stdcall *)(void*)) java_start,
	                           thread,
	                           CREATE_SUSPENDED,
	                           &thread_id);
	  }

  {- -------------------------------------------
  (1) 以上の処理でスレッド生成に成功しなかった場合は, 
      生成したリソースを解放し, ここでリターン.
      ---------------------------------------- -}

	  if (thread_handle == NULL) {
	    // Need to clean up stuff we've allocated so far
	    CloseHandle(osthread->interrupt_event());
	    thread->set_osthread(NULL);
	    delete osthread;
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) os::win32::_os_thread_count フィールドの値を増加させておく.
      (この値は何に使われる?? 現在の実装では意味のある使われ方をしていないような... #TODO)
      ---------------------------------------- -}

	  Atomic::inc_ptr((intptr_t*)&os::win32::_os_thread_count);
	
  {- -------------------------------------------
  (1) OSThread::set_thread_handle() 及び OSThread::set_thread_id() を呼んで, 
      生成したスレッドの HANDLE 及び thread ID を OSThread オブジェクトに対応づける.
      ---------------------------------------- -}

	  // Store info on the Win32 thread into the OSThread
	  osthread->set_thread_handle(thread_handle);
	  osthread->set_thread_id(thread_id);
	
  {- -------------------------------------------
  (1) 生成したスレッドの ThreadState を INITIALIZED に変更する.
      ---------------------------------------- -}

	  // Initial thread state is INITIALIZED, not SUSPENDED
	  osthread->set_state(INITIALIZED);
	
  {- -------------------------------------------
  (1) リターン
      (この段階では, 生成したスレッドは INITIALIZED 状態で有り, 稼働はしていない(suspend されている状態).
       スレッドの開始処理は, ここからリターンした後の関数によって行われる.)
      ---------------------------------------- -}

	  // The thread is returned suspended (in state INITIALIZED), and is started higher up in the call chain
	  return true;
	}
	
```


