---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/stubGenerator_sparc.cpp
### 説明(description)

```
  //----------------------------------------------------------------------------------------------------
  // Continuation point for runtime calls returning with a pending exception
  // The pending exception check happened in the runtime or native call stub
  // The pending exception in Thread is converted into a Java-level exception
  //
  // Contract with Java-level exception handler: O0 = exception
  //                                             O1 = throwing pc

```

### 名前(function name)
```
  address generate_forward_exception() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下, StubCodeMark によるスタブ生成を行う) (See: StubCodeMark)
      ---------------------------------------- -}

	    StubCodeMark mark(this, "StubRoutines", "forward_exception");

  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	    address start = __ pc();
	
  {- -------------------------------------------
  (1) (このスタブに入ってきた時点では, O7 に 
       Java コードへのリターンアドレス(インタープリタ or JIT 生成コード)が入っている.
       例外処理ルーチンからのリターンアドレスなので, つまり例外送出が起こった PC が O7 に入っている.)
      ---------------------------------------- -}

	    // Upon entry, O7 has the return address returning into Java
	    // (interpreted or compiled) code; i.e. the return address
	    // becomes the throwing pc.
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    const Register& handler_reg = Gtemp;
	
	    Address exception_addr(G2_thread, Thread::pending_exception_offset());
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	    // make sure that this code is only executed if there is a pending exception
	    { Label L;
	      __ ld_ptr(exception_addr, Gtemp);
	      __ br_notnull(Gtemp, false, Assembler::pt, L);
	      __ delayed()->nop();
	      __ stop("StubRoutines::forward exception: no pending exception (1)");
	      __ bind(L);
	    }
	#endif
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの pending_exception フィールドから exception オブジェクトを取得し, Oexception レジスタに入れておく.
        (これは, SharedRuntime::exception_handler_for_return_address() の引数として使う)」
      ---------------------------------------- -}

	    // compute exception handler into handler_reg
	    __ get_thread();
	    __ ld_ptr(exception_addr, Oexception);

  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	    __ verify_oop(Oexception);

  {- -------------------------------------------
  (1) コード生成:
      「SharedRuntime::exception_handler_for_return_address() を呼んで, 対応する例外ハンドラのアドレスを取得する.
        (呼び出しの前後では save/restore も行っている)」
  
      (なお, SharedRuntime::exception_handler_for_return_address() は, 
       例外発生時の PC が Lscratch に入っていると想定している模様)
      ---------------------------------------- -}

	    __ save_frame(0);             // compensates for compiler weakness
	    __ add(O7->after_save(), frame::pc_return_offset, Lscratch); // save the issuing PC
	    BLOCK_COMMENT("call exception_handler_for_return_address");
	    __ call_VM_leaf(L7_thread_cache, CAST_FROM_FN_PTR(address, SharedRuntime::exception_handler_for_return_address), G2_thread, Lscratch);
	    __ mov(O0, handler_reg);
	    __ restore();                 // compensates for compiler weakness
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの pending_exception フィールドから exception オブジェクトを取得し, Oexception レジスタに入れておく.
        また, 例外発生時の PC を Oissuing_pc に入れておく.」
      ---------------------------------------- -}

	    __ ld_ptr(exception_addr, Oexception);
	    __ add(O7, frame::pc_return_offset, Oissuing_pc); // save the issuing PC
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	    // make sure exception is set
	    { Label L;
	      __ br_notnull(Oexception, false, Assembler::pt, L);
	      __ delayed()->nop();
	      __ stop("StubRoutines::forward exception: no pending exception (2)");
	      __ bind(L);
	    }
	#endif

  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの pending_exception フィールドをクリアした後, 取得した例外ハンドラのアドレスにジャンプする」
      ---------------------------------------- -}

	    // jump to exception handler
	    __ jmp(handler_reg, 0);
	    // clear pending exception
	    __ delayed()->st_ptr(G0, exception_addr);
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン
      ---------------------------------------- -}

	    return start;
	  }
	
```


