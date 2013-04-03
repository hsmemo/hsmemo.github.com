---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp
### 説明(description)

```
// Allocate monitor and lock method (asm interpreter)
// ebx - methodOop
//
```

### 名前(function name)
```
void InterpreterGenerator::lock_method(void) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「Lmethod から access flags を取得」
      ---------------------------------------- -}

	  __ ld(Lmethod, in_bytes(methodOopDesc::access_flags_offset()), O0);  // Load access flags.
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	 { Label ok;
	   __ btst(JVM_ACC_SYNCHRONIZED, O0);
	   __ br( Assembler::notZero, false, Assembler::pt, ok);
	   __ delayed()->nop();
	   __ stop("method doesn't need synchronization");
	   __ bind(ok);
	  }
	#endif // ASSERT
	
  {- -------------------------------------------
  (1) コード生成:
      「access flags から static かどうかをチェックする.
       チェック結果に応じて, 処理が ２通りに分岐.」
      ---------------------------------------- -}

	  // get synchronization object to O0
	  { Label done;
	    const int mirror_offset = klassOopDesc::klass_part_offset_in_bytes() + Klass::java_mirror_offset_in_bytes();
	    __ btst(JVM_ACC_STATIC, O0);

  {- -------------------------------------------
  (1) コード生成:
      「static でない場合は, Llocals から this オブジェクトを取得」
      ---------------------------------------- -}

	    __ br( Assembler::zero, true, Assembler::pt, done);
	    __ delayed()->ld_ptr(Llocals, Interpreter::local_offset_in_bytes(0), O0); // get receiver for not-static case
	
  {- -------------------------------------------
  (1) コード生成:
      「static の場合は, Lmethod から mirror オブジェクト (= Java レベルでのクラスオブジェクト) を取得」
      ---------------------------------------- -}

	    __ ld_ptr( Lmethod, in_bytes(methodOopDesc::constants_offset()), O0);
	    __ ld_ptr( O0, constantPoolOopDesc::pool_holder_offset_in_bytes(), O0);
	
	    // lock the mirror, not the klassOop
	    __ ld_ptr( O0, mirror_offset, O0);
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	    __ tst(O0);
	    __ breakpoint_trap(Assembler::zero);
	#endif // ASSERT
	
  {- -------------------------------------------
  (1) コード生成:
      「(static/非static どちらの場合もここで合流)」
      ---------------------------------------- -}

	    __ bind(done);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::add_monitor_to_stack() が生成するコードで
      スタックフレーム上に BasicObjectLock を確保する」
      (なお, stack_is_empty 引数は true で呼び出している)
      ---------------------------------------- -}

	  __ add_monitor_to_stack(true, noreg, noreg);  // allocate monitor elem

  {- -------------------------------------------
  (1) コード生成:
      「BasicObjectLock にロック対象のオブジェクトの oop を書き込む」
      ---------------------------------------- -}

	  __ st_ptr( O0, Lmonitors, BasicObjectLock::obj_offset_in_bytes());   // store object

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::lock_object() が生成するコードで, 
       対象のオブジェクトにロック処理を行う」
      ---------------------------------------- -}

	  // __ untested("lock_object from method entry");
	  __ lock_object(Lmonitors, O0);
	
```


