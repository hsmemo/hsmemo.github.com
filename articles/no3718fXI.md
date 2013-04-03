---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/jniHandles.cpp

### 名前(function name)
```
JNIHandleBlock* JNIHandleBlock::allocate_block(Thread* thread)  {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(thread == NULL || thread == Thread::current(), "sanity check");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JNIHandleBlock* block;

  {- -------------------------------------------
  (1) もし thread local な空き領域から確保できるのであれば, そこから確保する.
      ---------------------------------------- -}

	  // Check the thread-local free list for a block so we don't
	  // have to acquire a mutex.
	  if (thread != NULL && thread->free_handle_block() != NULL) {
	    block = thread->free_handle_block();
	    thread->set_free_handle_block(block->_next);
	  }

  {- -------------------------------------------
  (1) もし thread local な空き領域からの確保ができなければ, 
      以下の確保処理を行う.
  
      (なお, この処理は JNIHandleBlockFreeList_lock で排他して行うが, 
       ロックする際に safepoint の確認は行わない.
       下記のコメントによると, 確認すると dead lock する恐れがあるため, とのこと.)
      ---------------------------------------- -}

	  else {
	    // locking with safepoint checking introduces a potential deadlock:
	    // - we would hold JNIHandleBlockFreeList_lock and then Threads_lock
	    // - another would hold Threads_lock (jni_AttachCurrentThread) and then
	    //   JNIHandleBlockFreeList_lock (JNIHandleBlock::allocate_block)
	    MutexLockerEx ml(JNIHandleBlockFreeList_lock,
	                     Mutex::_no_safepoint_check_flag);

    {- -------------------------------------------
  (1.1) もし JNIHandleBlock::_block_free_list にも空きブロックがなければ, 
        新しい JNIHandleBlock オブジェクトを new して確保する.
        
        (なお, JNIHandleBlock::_block_free_list は
         一度使われた JNIHandleBlock を解放された後もすぐ再利用できるように free list でつないだもの.
         (See: JNIHandleBlock::release_block()))
        ---------------------------------------- -}

	    if (_block_free_list == NULL) {
	      // Allocate new block
	      block = new JNIHandleBlock();
	      _blocks_allocated++;

      {- -------------------------------------------
  (1.1.1) (トレース出力)
          ---------------------------------------- -}

	      if (TraceJNIHandleAllocation) {
	        tty->print_cr("JNIHandleBlock " INTPTR_FORMAT " allocated (%d total blocks)",
	                      block, _blocks_allocated);
	      }

      {- -------------------------------------------
  (1.1.1) (デバッグ用の処理) (関連する develop オプションが指定されている場合にのみ実行) (See: ZapJNIHandleArea)
          (JNIHandleBlock::zap() を呼び出して中身を破壊しておく)
          ---------------------------------------- -}

	      if (ZapJNIHandleArea) block->zap();

      {- -------------------------------------------
  (1.1.1) (デバッグ用の処理) (#ifndef PRODUCT 時にのみ実行)
          ---------------------------------------- -}

	      #ifndef PRODUCT
	      // Link new block to list of all allocated blocks
	      block->_block_list_link = _block_list;
	      _block_list = block;
	      #endif

    {- -------------------------------------------
  (1.1) もし JNIHandleBlock::_block_free_list に空きブロックがあれば,
        そこから JNIHandleBlock を取得.
        ---------------------------------------- -}

	    } else {
	      // Get block from free list
	      block = _block_free_list;
	      _block_free_list = _block_free_list->_next;
	    }
	  }

  {- -------------------------------------------
  (1) 確保できた JNIHandleBlock オブジェクトの初期化を行う.
      ---------------------------------------- -}

	  block->_top  = 0;
	  block->_next = NULL;
	  block->_pop_frame_link = NULL;

  {- -------------------------------------------
  (1) (デバッグ用の処理) (debug_only 時にのみ実行)
      確保できた JNIHandleBlock オブジェクトの初期化を行う.
      (なお, _last, _free_list, 及び _allocate_before_rebuild フィールドについては
       JNIHandleBlock::allocate_handle() メソッド内で初期化している.
       See: JNIHandleBlock::allocate_handle())
      ---------------------------------------- -}

	  // _last, _free_list & _allocate_before_rebuild initialized in allocate_handle
	  debug_only(block->_last = NULL);
	  debug_only(block->_free_list = NULL);
	  debug_only(block->_allocate_before_rebuild = -1);

  {- -------------------------------------------
  (1) 結果をリターン.
      ---------------------------------------- -}

	  return block;
	}
	
```


