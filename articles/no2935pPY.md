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
static void generate_satb_log_enqueue(bool with_frame) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) BufferBlob::create() で BufferBlob を生成する.
      ---------------------------------------- -}

	  BufferBlob* bb = BufferBlob::create("enqueue_with_frame", EnqueueCodeSize);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  CodeBuffer buf(bb);
	  MacroAssembler masm(&buf);
	  address start = masm.pc();
	  Register pre_val;
	
	  Label refill, restart;

  {- -------------------------------------------
  (1) コード生成: (ただし, with_frame 引数が true の場合にのみ生成)
      「MacroAssembler::save_frame() が生成するコードにより, レジスタを待避しておく.」
  
       (なお, 値は O0 で渡されてくる. そのため save すると処理対象は I0 になる.
        See: MacroAssembler::g1_write_barrier_pre())
      ---------------------------------------- -}

	  if (with_frame) {
	    masm.save_frame(0);
	    pre_val = I0;  // Was O0 before the save.
	  } else {
	    pre_val = O0;
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int satb_q_index_byte_offset =
	    in_bytes(JavaThread::satb_mark_queue_offset() +
	             PtrQueue::byte_offset_of_index());
	  int satb_q_buf_byte_offset =
	    in_bytes(JavaThread::satb_mark_queue_offset() +
	             PtrQueue::byte_offset_of_buf());

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(in_bytes(PtrQueue::byte_width_of_index()) == sizeof(intptr_t) &&
	         in_bytes(PtrQueue::byte_width_of_buf()) == sizeof(intptr_t),
	         "check sizes in assembly below");
	
  {- -------------------------------------------
  (1) (ここが restart ラベルの位置. refill 処理を行った場合に処理をやり直す地点)
      ---------------------------------------- -}

	  masm.bind(restart);

  {- -------------------------------------------
  (1) コード生成:
      「JavaThread::_satb_mark_queue フィールドの ObjPtrQueue オブジェクト内の
        _index フィールドを L0 レジスタにロードする.
        (ロード結果が 0 の場合は, 次のバッファを用意する必要があるので, refill ラベルにジャンプする).」
    
       (なお, _index がバッファ中の現在の終端位置を示す. 値が 0 であればもう空き領域はない)
      ---------------------------------------- -}

	  masm.ld_ptr(G2_thread, satb_q_index_byte_offset, L0);
	
	  masm.br_on_reg_cond(Assembler::rc_z, /*annul*/false, Assembler::pn, L0, refill);

  {- -------------------------------------------
  (1) コード生成:
      「JavaThread::_satb_mark_queue フィールドの ObjPtrQueue オブジェクト内の
        _buf フィールドをロードする (なおこのロードは遅延スロットで実行される. ジャンプ先では使われないが害もない).」
      ---------------------------------------- -}

	  // If the branch is taken, no harm in executing this in the delay slot.
	  masm.delayed()->ld_ptr(G2_thread, satb_q_buf_byte_offset, L1);

  {- -------------------------------------------
  (1) コード生成:
      「L0 レジスタの値を oopSize 分だけ小さくしておく.」      
      ---------------------------------------- -}

	  masm.sub(L0, oopSize, L0);
	
  {- -------------------------------------------
  (1) コード生成:
      「JavaThread::_satb_mark_queue フィールドの ObjPtrQueue オブジェクト内の
       _buf 内の "_index - oopSize" の位置に保存対象の値(O0/I0レジスタの値)を書き込む.
       (より具体的に言うと, L0 と L1 を足したアドレスに値を書き込む)」
      ---------------------------------------- -}

	  masm.st_ptr(pre_val, L1, L0);  // [_buf + index] := I0

  {- -------------------------------------------
  (1) コード生成:
      「JavaThread::_satb_mark_queue フィールドの ObjPtrQueue オブジェクト内の
        _index フィールドに, 新しい値(oopSize 分だけ小さくした値)を書き戻す.
        その後, リターンする.
  
        (正確には with_frame 引数が false であれば, 書き込み処理はリターンの遅延スロットで行われる.
        また, with_frame 引数が true の場合には, restore 処理も行われる)」
      ---------------------------------------- -}

	  if (!with_frame) {
	    // Use return-from-leaf
	    masm.retl();
	    masm.delayed()->st_ptr(L0, G2_thread, satb_q_index_byte_offset);
	  } else {
	    // Not delayed.
	    masm.st_ptr(L0, G2_thread, satb_q_index_byte_offset);
	  }
	  if (with_frame) {
	    masm.ret();
	    masm.delayed()->restore();
	  }

  {- -------------------------------------------
  (1) (以降が refill ラベルに飛んだ場合の処理)
      ---------------------------------------- -}

	  masm.bind(refill);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  address handle_zero =
	    CAST_FROM_FN_PTR(address,
	                     &SATBMarkQueueSet::handle_zero_index_for_thread);

  {- -------------------------------------------
  (1) (コメントによると
       このケースは稀なので, 使用されている可能性のある全てのスクラッチレジスタを
       ここで待避する程度は問題ないと想定している.
       とのこと)
      ---------------------------------------- -}

	  // This should be rare enough that we can afford to save all the
	  // scratch registers that the calling context might be using.

  {- -------------------------------------------
  (1) コード生成:
      「スクラッチ用の G レジスタを待避」
      ---------------------------------------- -}

	  masm.mov(G1_scratch, L0);
	  masm.mov(G3_scratch, L1);
	  masm.mov(G4, L2);

  {- -------------------------------------------
  (1) コード生成:
      「O0 レジスタの値が消えると困るので待避しておく.
        また, O7 レジスタの値も消えると困るので待避しておく」
      ---------------------------------------- -}

	  // We need the value of O0 above (for the write into the buffer), so we
	  // save and restore it.
	  masm.mov(O0, L3);
	  // Since the call will overwrite O7, we save and restore that, as well.
	  masm.mov(O7, L4);

  {- -------------------------------------------
  (1) コード生成:
      「MacroAssembler::call_VM_leaf() が生成するコードにより, 
        SATBMarkQueueSet::handle_zero_index_for_thread() を呼び出す.」
      ---------------------------------------- -}

	  masm.call_VM_leaf(L5, handle_zero, G2_thread);

  {- -------------------------------------------
  (1) コード生成:
      「待避していたレジスタを復帰し, restart ラベルに戻って処理をやり直す.」
      ---------------------------------------- -}

	  masm.mov(L0, G1_scratch);
	  masm.mov(L1, G3_scratch);
	  masm.mov(L2, G4);
	  masm.mov(L3, O0);
	  masm.br(Assembler::always, /*annul*/false, Assembler::pt, restart);
	  masm.delayed()->mov(L4, O7);
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスと終端アドレスを
      対応する大域変数にセットする.
      ---------------------------------------- -}

	  if (with_frame) {
	    satb_log_enqueue_with_frame = start;
	    satb_log_enqueue_with_frame_end = masm.pc();
	  } else {
	    satb_log_enqueue_frameless = start;
	    satb_log_enqueue_frameless_end = masm.pc();
	  }
	}
	
```


