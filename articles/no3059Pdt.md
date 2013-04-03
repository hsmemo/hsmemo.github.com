---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/stubGenerator_x86_64.cpp
### 説明(description)

```
  // Return point for a Java call if there's an exception thrown in
  // Java code.  The exception is caught and transformed into a
  // pending exception stored in JavaThread that can be tested from
  // within the VM.
  //
  // Note: Usually the parameters are removed by the callee. In case
  // of an exception crossing an activation frame boundary, that is
  // not the case if the callee is compiled code => need to setup the
  // rsp.
  //
  // rax: exception oop

```

### 名前(function name)
```
  address generate_catch_exception() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下, StubCodeMark によるスタブ生成を行う) (See: StubCodeMark)
      ---------------------------------------- -}

	    StubCodeMark mark(this, "StubRoutines", "catch_exception");

  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	    address start = __ pc();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (ところで, rsp_after_call の方は使われていないような...
       thread の方も #ifdef ASSERT 時にしか使われていないような... #TODO)
      ---------------------------------------- -}

	    // same as in generate_call_stub():
	    const Address rsp_after_call(rbp, rsp_after_call_off * wordSize);
	    const Address thread        (rbp, thread_off         * wordSize);
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	    // verify that threads correspond
	    {
	      Label L, S;
	      __ cmpptr(r15_thread, thread);
	      __ jcc(Assembler::notEqual, S);
	      __ get_thread(rbx);
	      __ cmpptr(r15_thread, rbx);
	      __ jcc(Assembler::equal, L);
	      __ bind(S);
	      __ stop("StubRoutines::catch_exception: threads must correspond");
	      __ bind(L);
	    }
	#endif
	
  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	    // set pending exception
	    __ verify_oop(rax);
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドを表す JavaThread に対して, 
        以下の各フィールドの値をセットする.
        * pending_exception フィールド
          rax レジスタに入っている値 (= 送出された例外オブジェクト)
        * exception_file フィールド
          __FILE__ の値 (= このファイルのファイル名)
        * exception_line フィールド
          __LINE__ の値 (= このコード自体の行番号)            」
      ---------------------------------------- -}

	    __ movptr(Address(r15_thread, Thread::pending_exception_offset()), rax);
	    __ lea(rscratch1, ExternalAddress((address)__FILE__));
	    __ movptr(Address(r15_thread, Thread::exception_file_offset()), rscratch1);
	    __ movl(Address(r15_thread, Thread::exception_line_offset()), (int)  __LINE__);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    // complete return to VM
	    assert(StubRoutines::_call_stub_return_address != NULL,
	           "_call_stub_return_address must have been generated before");

  {- -------------------------------------------
  (1) コード生成:
      「StubRoutines::_call_stub_return_address にジャンプする」
      ---------------------------------------- -}

	    __ jump(RuntimeAddress(StubRoutines::_call_stub_return_address));
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン
      ---------------------------------------- -}

	    return start;
	  }
	
```


