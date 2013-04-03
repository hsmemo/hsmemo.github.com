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
  // Return point for a Java call if there's an exception thrown in Java code.
  // The exception is caught and transformed into a pending exception stored in
  // JavaThread that can be tested from within the VM.
  //
  // Oexception: exception oop

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
  (1) コード生成: (verify)
      ---------------------------------------- -}

	    // verify that thread corresponds
	    __ verify_thread();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    const Register& temp_reg = Gtemp;
	    Address pending_exception_addr    (G2_thread, Thread::pending_exception_offset());
	    Address exception_file_offset_addr(G2_thread, Thread::exception_file_offset   ());
	    Address exception_line_offset_addr(G2_thread, Thread::exception_line_offset   ());
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドを表す JavaThread に対して, 
        以下の各フィールドの値をセットする.
        * pending_exception フィールド
          Oexception レジスタに入っている値 (= 送出された例外オブジェクト)
        * exception_file フィールド
          __FILE__ の値 (= このファイルのファイル名)
        * exception_line フィールド
          __LINE__ の値 (= このコード自体の行番号)                   」
      ---------------------------------------- -}

	    // set pending exception
	    __ verify_oop(Oexception);
	    __ st_ptr(Oexception, pending_exception_addr);
	    __ set((intptr_t)__FILE__, temp_reg);
	    __ st_ptr(temp_reg, exception_file_offset_addr);
	    __ set((intptr_t)__LINE__, temp_reg);
	    __ st(temp_reg, exception_line_offset_addr);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    // complete return to VM
	    assert(StubRoutines::_call_stub_return_address != NULL, "must have been generated before");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    AddressLiteral stub_ret(StubRoutines::_call_stub_return_address);

  {- -------------------------------------------
  (1) コード生成:
      「StubRoutines::_call_stub_return_address にジャンプする」
      ---------------------------------------- -}

	    __ jump_to(stub_ret, temp_reg);
	    __ delayed()->nop();
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン
      ---------------------------------------- -}

	    return start;
	  }
	
```


