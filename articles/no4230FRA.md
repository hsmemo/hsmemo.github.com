---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/assembler_sparc.cpp

### 名前(function name)
```
void MacroAssembler::set_last_Java_frame(Register last_java_sp, Register last_Java_pc) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_not_delayed();

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Address flags(G2_thread, JavaThread::frame_anchor_offset() +
	                           JavaFrameAnchor::flags_offset());
	  Address pc_addr(G2_thread, JavaThread::last_Java_pc_offset());
	
  {- -------------------------------------------
  (1) (last_Java_sp への書き込みは最後に行う.
       これは, last_Java_sp に値が書き込まれている状態では他のフィールドにも値が入っている, と保証するため)
      ---------------------------------------- -}

	  // Always set last_Java_pc and flags first because once last_Java_sp is visible
	  // has_last_Java_frame is true and users will look at the rest of the fields.
	  // (Note: flags should always be zero before we get here so doesn't need to be set.)
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	  // Verify that flags was zeroed on return to Java
	  Label PcOk;
	  save_frame(0);                // to avoid clobbering O0
	  ld_ptr(pc_addr, L0);
	  tst(L0);
	#ifdef _LP64
	  brx(Assembler::zero, false, Assembler::pt, PcOk);
	#else
	  br(Assembler::zero, false, Assembler::pt, PcOk);
	#endif // _LP64
	  delayed() -> nop();
	  stop("last_Java_pc not zeroed before leaving Java");
	  bind(PcOk);
	
	  // Verify that flags was zeroed on return to Java
	  Label FlagsOk;
	  ld(flags, L0);
	  tst(L0);
	  br(Assembler::zero, false, Assembler::pt, FlagsOk);
	  delayed() -> restore();
	  stop("flags not zeroed before leaving Java");
	  bind(FlagsOk);
	#endif /* ASSERT */
	  //
	  // When returning from calling out from Java mode the frame anchor's last_Java_pc
	  // will always be set to NULL. It is set here so that if we are doing a call to
	  // native (not VM) that we capture the known pc and don't have to rely on the
	  // native call having a standard frame linkage where we can find the pc.
	
  {- -------------------------------------------
  (1) コード生成: (ただし, 引数で last_java_pc が指定されていなければ生成しない)
      「カレントスレッドの last_Java_pc フィールドに, 指定されたレジスタの値をセットする.」
      ---------------------------------------- -}

	  if (last_Java_pc->is_valid()) {
	    st_ptr(last_Java_pc, pc_addr);
	  }
	
  {- -------------------------------------------
  (1) (以下のコードは _LP64 かどうかで2通り存在)
      ---------------------------------------- -}

	#ifdef _LP64

  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	  // Make sure that we have an odd stack
	  Label StackOk;
	  andcc(last_java_sp, 0x01, G0);
	  br(Assembler::notZero, false, Assembler::pt, StackOk);
	  delayed() -> nop();
	  stop("Stack Not Biased in set_last_Java_frame");
	  bind(StackOk);
	#endif // ASSERT
	  assert( last_java_sp != G4_scratch, "bad register usage in set_last_Java_frame");

  {- -------------------------------------------
  (1) コード生成: (こちらは _LP64 環境用)
      「last_java_sp を STACK_BIAS 分だけ調整した後, 
        カレントスレッドの last_Java_sp フィールドにセットする.」
      ---------------------------------------- -}

	  add( last_java_sp, STACK_BIAS, G4_scratch );
	  st_ptr(G4_scratch, G2_thread, JavaThread::last_Java_sp_offset());
	#else

  {- -------------------------------------------
  (1) コード生成: (こちらは 非_LP64 環境用)
      「カレントスレッドの last_Java_sp フィールドをセットする.」
      ---------------------------------------- -}

	  st_ptr(last_java_sp, G2_thread, JavaThread::last_Java_sp_offset());
	#endif // _LP64
	
```


