---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_64.cpp
### 説明(description)

```
// Allocate monitor and lock method (asm interpreter)
//
// Args:
//      rbx: methodOop
//      r14: locals
//
// Kills:
//      rax
//      c_rarg0, c_rarg1, c_rarg2, c_rarg3, ...(param regs)
//      rscratch1, rscratch2 (scratch regs)
```

### 名前(function name)
```
void InterpreterGenerator::lock_method(void) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // synchronize method
	  const Address access_flags(rbx, methodOopDesc::access_flags_offset());
	  const Address monitor_block_top(
	        rbp,
	        frame::interpreter_frame_monitor_block_top_offset * wordSize);
	  const int entry_size = frame::interpreter_frame_monitor_size() * wordSize;
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      (access flags の synchronized ビットが立っていることをチェックしている)
      ---------------------------------------- -}

	#ifdef ASSERT
	  {
	    Label L;
	    __ movl(rax, access_flags);
	    __ testl(rax, JVM_ACC_SYNCHRONIZED);
	    __ jcc(Assembler::notZero, L);
	    __ stop("method doesn't need synchronization");
	    __ bind(L);
	  }
	#endif // ASSERT
	
  {- -------------------------------------------
  (1) コード生成:
      「rbx から access flags を取得し, static かどうかをチェックする.
       チェック結果に応じて, 処理が ２通りに分岐.」
      ---------------------------------------- -}

	  // get synchronization object
	  {
	    const int mirror_offset = klassOopDesc::klass_part_offset_in_bytes() +
	                              Klass::java_mirror_offset_in_bytes();
	    Label done;
	    __ movl(rax, access_flags);
	    __ testl(rax, JVM_ACC_STATIC);

  {- -------------------------------------------
  (1) コード生成:
      「static でない場合は, スタックフレーム内に待避していた locals から this オブジェクトを取得」
      ---------------------------------------- -}

	    // get receiver (assume this is frequent case)
	    __ movptr(rax, Address(r14, Interpreter::local_offset_in_bytes(0)));
	    __ jcc(Assembler::zero, done);

  {- -------------------------------------------
  (1) コード生成:
      「static の場合は, スタックフレーム内に待避していた methodOopDesc 中から 
       mirror オブジェクト (= Java レベルでのクラスオブジェクト) を取得」  
      ---------------------------------------- -}

	    __ movptr(rax, Address(rbx, methodOopDesc::constants_offset()));
	    __ movptr(rax, Address(rax,
	                           constantPoolOopDesc::pool_holder_offset_in_bytes()));
	    __ movptr(rax, Address(rax, mirror_offset));
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	    {
	      Label L;
	      __ testptr(rax, rax);
	      __ jcc(Assembler::notZero, L);
	      __ stop("synchronization object is NULL");
	      __ bind(L);
	    }
	#endif // ASSERT
	
  {- -------------------------------------------
  (1) コード生成:
      「(static/非static どちらの場合もここで合流)」
      ---------------------------------------- -}

	    __ bind(done);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「stack frame 上に BasicObjectLock を確保する」
      ---------------------------------------- -}

	  // add space for monitor & lock
	  __ subptr(rsp, entry_size); // add space for a monitor entry
	  __ movptr(monitor_block_top, rsp);  // set new monitor block top

  {- -------------------------------------------
  (1) コード生成:
      「BasicObjectLock にロック対象のオブジェクトの oop を書き込む」
      ---------------------------------------- -}

	  // store object
	  __ movptr(Address(rsp, BasicObjectLock::obj_offset_in_bytes()), rax);
	  __ movptr(c_rarg1, rsp); // object address

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::lock_object() が生成するコードで, 
       対象のオブジェクトにロック処理を行う」
      ---------------------------------------- -}

	  __ lock_object(c_rarg1);
	
```


