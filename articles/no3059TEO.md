---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp

### 名前(function name)
```
void TemplateInterpreterGenerator::generate_stack_overflow_check(Register Rframe_size,
                                                         Register Rscratch,
                                                         Register Rscratch2) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      (page_size は, 実行環境におけるメモリページ 1 枚の大きさ)
      ---------------------------------------- -}

	  const int page_size = os::vm_page_size();
	  Address saved_exception_pc(G2_thread, JavaThread::saved_exception_pc_offset());
	  Label after_frame_check;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(Rframe_size, Rscratch, Rscratch2);
	
  {- -------------------------------------------
  (1) コード生成:
      「まずは, 引数で指定されたフレームサイズ(Rframe_size)が, 1メモリページ(page_size)を超えるかどうかをチェック.
        もしフレームサイズが 1ページ以下であれば, スタックオーバーフローの心配は無いので, 
        after_frame_check ラベルまでジャンプ. 」
      ---------------------------------------- -}

	  __ set( page_size,   Rscratch );
	  __ cmp( Rframe_size, Rscratch );
	
	  __ br( Assembler::lessEqual, false, Assembler::pt, after_frame_check );
	  __ delayed()->nop();
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドのスタックの底にあたるアドレス(数字的には一番大きなアドレス)を取得」
      ---------------------------------------- -}

	  // get the stack base, and in debug, verify it is non-zero
	  __ ld_ptr( G2_thread, Thread::stack_base_offset(), Rscratch );

  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	  Label base_not_zero;
	  __ cmp( Rscratch, G0 );
	  __ brx( Assembler::notEqual, false, Assembler::pn, base_not_zero );
	  __ delayed()->nop();
	  __ stop("stack base is zero in generate_stack_overflow_check");
	  __ bind(base_not_zero);
	#endif
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドのスタックの長さを取得」
      ---------------------------------------- -}

	  // get the stack size, and in debug, verify it is non-zero
	  assert( sizeof(size_t) == sizeof(intptr_t), "wrong load size" );
	  __ ld_ptr( G2_thread, Thread::stack_size_offset(), Rscratch2 );

  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	  Label size_not_zero;
	  __ cmp( Rscratch2, G0 );
	  __ brx( Assembler::notEqual, false, Assembler::pn, size_not_zero );
	  __ delayed()->nop();
	  __ stop("stack size is zero in generate_stack_overflow_check");
	  __ bind(size_not_zero);
	#endif
	
  {- -------------------------------------------
  (1) コード生成:
      「yellow page 直前のアドレスを計算する」
  
      (stack_bottom - stack_length) + StackRedPages + StackYellowPages
      ---------------------------------------- -}

	  // compute the beginning of the protected zone minus the requested frame size
	  __ sub( Rscratch, Rscratch2,   Rscratch );
	  __ set( (StackRedPages+StackYellowPages) * page_size, Rscratch2 );
	  __ add( Rscratch, Rscratch2,   Rscratch );
	
  {- -------------------------------------------
  (1) コード生成:
      「もし, yellow page 直前のアドレスと現在の SP の間が
        要求されたフレームサイズ分よりも広ければ, 
        スタックオーバーフローの心配は無いので after_frame_check ラベルまでジャンプ.」
      ---------------------------------------- -}

	  // Add in the size of the frame (which is the same as subtracting it from the
	  // SP, which would take another register
	  __ add( Rscratch, Rframe_size, Rscratch );
	
	  // the frame is greater than one page in size, so check against
	  // the bottom of the stack
	  __ cmp( SP, Rscratch );
	  __ brx( Assembler::greater, false, Assembler::pt, after_frame_check );
	  __ delayed()->nop();
	
  {- -------------------------------------------
  (1) コード生成:
      「(十分な空き領域がないと分かったので) StackOverflowError を送出する.
  
        (なお送出する前に, 現在のリターンアドレスを
        カレントスレッドの JavaThread::saved_exception_pc フィールドに待避している)」
      ---------------------------------------- -}

	  // Save the return address as the exception pc
	  __ st_ptr(O7, saved_exception_pc);
	
	  // the stack will overflow, throw an exception
	  __ call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::throw_StackOverflowError));
	
  {- -------------------------------------------
  (1) コード生成:
      「(ここに到達したら, スタックには十分な空き領域があるということ)」
      ---------------------------------------- -}

	  // if you get to here, then there is enough stack space
	  __ bind( after_frame_check );
	}
	
```


