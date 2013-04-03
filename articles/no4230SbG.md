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
void MacroAssembler::reset_last_Java_frame(void) {
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

	  Address sp_addr(G2_thread, JavaThread::last_Java_sp_offset());
	  Address pc_addr(G2_thread, JavaThread::frame_anchor_offset() + JavaFrameAnchor::last_Java_pc_offset());
	  Address flags  (G2_thread, JavaThread::frame_anchor_offset() + JavaFrameAnchor::flags_offset());
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	  // check that it WAS previously set
	#ifdef CC_INTERP
	    save_frame(0);
	#else
	    save_frame_and_mov(0, Lmethod, Lmethod);     // Propagate Lmethod to helper frame for -Xprof
	#endif /* CC_INTERP */
	    ld_ptr(sp_addr, L0);
	    tst(L0);
	    breakpoint_trap(Assembler::zero, Assembler::ptr_cc);
	    restore();
	#endif // ASSERT
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの last_Java_sp フィールドをクリアする.」
  
      (なお, last_Java_sp のクリアを最初に行う.
       これは, last_Java_sp に値が書き込まれている状態では他のフィールドにも値が入っている, と保証するため)
      ---------------------------------------- -}

	  st_ptr(G0, sp_addr);

  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの last_Java_pc フィールドをクリアする.」
      ---------------------------------------- -}

	  // Always return last_Java_pc to zero
	  st_ptr(G0, pc_addr);

  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの flags フィールドをクリアする.」
      ---------------------------------------- -}

	  // Always null flags after return to Java
	  st(G0, flags);
	
```


