---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.cpp

### 名前(function name)
```
void ConcurrentGCThread::initialize_in_thread() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Thread::record_stack_base_and_size() を呼んで, スタック領域関連のフィールドを初期化する.
      ---------------------------------------- -}

	  this->record_stack_base_and_size();

  {- -------------------------------------------
  (1) Thread::initialize_thread_local_storage() を呼んで, 
      現在実行中のネイティブスレッドにこの JavaThread オブジェクトを対応付けておく.
      ---------------------------------------- -}

	  this->initialize_thread_local_storage();

  {- -------------------------------------------
  (1) カレントスレッドの JNI ローカル参照フレームを作成しておく.
      (JNIHandleBlock::allocate_block() で新しい JNIHandleBlock を作成し, 
       それを Thread::set_active_handles() でカレントスレッドにセットする.
       See: [here](no3059hRF.html) for details)
      ---------------------------------------- -}

	  this->set_active_handles(JNIHandleBlock::allocate_block());

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // From this time Thread::current() should be working.
	  assert(this == Thread::current(), "just checking");
	}
	
```


