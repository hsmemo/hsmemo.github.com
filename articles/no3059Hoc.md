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
// check if it's safe to start a new thread
```

### 名前(function name)
```
static bool _thread_safety_check(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下の値をリターンする.
      * 使用しているスレッドが Fixed stack LinuxThread であり, かつ ThreadSafetyMargin オプションの値も 0 ではない場合:
        現状のスタック領域の一番下のアドレス(以下の stack_bottom)と
        ヒープとして使っている領域の一番上のアドレス(以下の highest_vm_reserved_address())を比較し, 
        その間が ThreadSafetyMargin 以上離れていれば true を返す.
        そうでなければ false を返す.
  
      * それ以外の場合:
        無条件で true を返す.
  
  
      (LinuxThread には, スタックを mmap の MAP_FIXED で確保する実装バージョン(Fixed stack)がある.
       この場合, ヒープはメモリ空間の下の方に確保され, 一方でスタックはメモリ空間の上の方から
       固定長分(大抵は2MB)ずつ MAP_FIXED で確保されていく.
  
       問題になるのは, MAP_FIXED は既に mmap() 済みの領域に対しても成功する (マッピングを上書きする) ので, 
       スレッドが非常に多い場合にはスタックがヒープとぶつかってヒープの一部を消してしまう恐れがあること.
       
       これを防ぐために, 現状のスタック領域の一番下のアドレスと
       ヒープとして使っている領域の一番上のアドレスを比較し, 
       その間にある程度の距離がなければエラーにすることにしている.
  
       この距離は ThreadSafetyMargin オプションで調整できる.
       また, ThreadSafetyMargin オプションが 0 の場合には, この処理全体が無効になる.)
  
      (なお, NPTL や Fixed stack ではない LinuxThread の場合には, 
       スタックが伸びすぎて既に mmap() した領域とぶつかるケースでは
       そもそも pthread_create() が成功しない.
       そのため, ここに来た時点でスタックは問題なく確保されていることになる.)
      ---------------------------------------- -}

	  if (os::Linux::is_LinuxThreads() && !os::Linux::is_floating_stack()) {
	    // Fixed stack LinuxThreads (SuSE Linux/x86, and some versions of Redhat)
	    //   Heap is mmap'ed at lower end of memory space. Thread stacks are
	    //   allocated (MAP_FIXED) from high address space. Every thread stack
	    //   occupies a fixed size slot (usually 2Mbytes, but user can change
	    //   it to other values if they rebuild LinuxThreads).
	    //
	    // Problem with MAP_FIXED is that mmap() can still succeed even part of
	    // the memory region has already been mmap'ed. That means if we have too
	    // many threads and/or very large heap, eventually thread stack will
	    // collide with heap.
	    //
	    // Here we try to prevent heap/stack collision by comparing current
	    // stack bottom with the highest address that has been mmap'ed by JVM
	    // plus a safety margin for memory maps created by native code.
	    //
	    // This feature can be disabled by setting ThreadSafetyMargin to 0
	    //
	    if (ThreadSafetyMargin > 0) {
	      address stack_bottom = os::current_stack_base() - os::current_stack_size();
	
	      // not safe if our stack extends below the safety margin
	      return stack_bottom - ThreadSafetyMargin >= highest_vm_reserved_address();
	    } else {
	      return true;
	    }
	  } else {
	    // Floating stack LinuxThreads or NPTL:
	    //   Unlike fixed stack LinuxThreads, thread stacks are not MAP_FIXED. When
	    //   there's not enough space left, pthread_create() will fail. If we come
	    //   here, that means enough space has been reserved for stack.
	    return true;
	  }
	}
	
```


