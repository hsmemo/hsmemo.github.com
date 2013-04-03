---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/assembler_sparc.cpp
### 説明(description)
(この関数が生成するコードは O0 レジスタにアドレスが入っていると想定している)

```
// This gets to assume that o0 contains the object address.
```

### 名前(function name)
```
static void generate_dirty_card_log_enqueue(jbyte* byte_map_base) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) BufferBlob::create() で BufferBlob を生成する.
      ---------------------------------------- -}

	  BufferBlob* bb = BufferBlob::create("dirty_card_enqueue", EnqueueCodeSize*2);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  CodeBuffer buf(bb);
	  MacroAssembler masm(&buf);
	  address start = masm.pc();
	
	  Label not_already_dirty, restart, refill;
	
  {- -------------------------------------------
  (1) コード生成: 
      「書き込み先のアドレスに対応する card table 中の値を確認する.
        まだ dirty (= 0) でなければ, not_already_dirty ラベルにジャンプして処理を行う.
        (既に dirty になっている場合は, このままフォールスルーしてすぐにリターンするだけ)
        (なお, ジャンプの遅延スロットで, card table 中の該当する箇所のアドレスを O3 レジスタにセットしている.
         これは dirty でなかった場合の処理で使用される)」
  
      (より具体的には, 
       まず, 書き込み先のアドレス(O0)の値を CardTableModRefBS::card_shift 分だけ右シフトさせて, 
       対応する card table 内の index を計算する.
       次に, それに card table のアドレス(byte_map_base 引数の値)を足し込んだアドレスから
       1byte ロードし, 0 と比較する)
  
      (なお, 0 が dirty を示す (See: dirty_card))
      ---------------------------------------- -}

	#ifdef _LP64
	  masm.srlx(O0, CardTableModRefBS::card_shift, O0);
	#else
	  masm.srl(O0, CardTableModRefBS::card_shift, O0);
	#endif
	  AddressLiteral addrlit(byte_map_base);
	  masm.set(addrlit, O1); // O1 := <card table base>
	  masm.ldub(O0, O1, O2); // O2 := [O0 + O1]
	
	  masm.br_on_reg_cond(Assembler::rc_nz, /*annul*/false, Assembler::pt,
	                      O2, not_already_dirty);
	  // Get O1 + O2 into a reg by itself -- useful in the take-the-branch
	  // case, harmless if not.
	  masm.delayed()->add(O0, O1, O3);
	
  {- -------------------------------------------
  (1) コード生成: 
      「リターンする」 (これは, 既に dirty だった場合の処理)
      ---------------------------------------- -}

	  // We didn't take the branch, so we're already dirty: return.
	  // Use return-from-leaf
	  masm.retl();
	  masm.delayed()->nop();
	
  {- -------------------------------------------
  (1) (以下が, まだ dirty ではなかった場合の処理)
      ---------------------------------------- -}

	  // Not dirty.
	  masm.bind(not_already_dirty);

  {- -------------------------------------------
  (1) コード生成: 
      「card table 中の該当箇所に 0 を書き込んで dirty 化する」
      ---------------------------------------- -}

	  // First, dirty it.
	  masm.stb(G0, O3, G0);  // [cardPtr] := 0  (i.e., dirty).

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int dirty_card_q_index_byte_offset =
	    in_bytes(JavaThread::dirty_card_queue_offset() +
	             PtrQueue::byte_offset_of_index());
	  int dirty_card_q_buf_byte_offset =
	    in_bytes(JavaThread::dirty_card_queue_offset() +
	             PtrQueue::byte_offset_of_buf());

  {- -------------------------------------------
  (1) (ここが restart ラベルの位置. refill 処理を行った場合に処理をやり直す地点)
      ---------------------------------------- -}

	  masm.bind(restart);

  {- -------------------------------------------
  (1) コード生成:
      「JavaThread::_dirty_card_queue フィールドの DirtyCardQueue オブジェクト内の
        _index フィールドを L0 レジスタにロードする.
        (ロード結果が 0 の場合は, 次のバッファを用意する必要があるので, refill ラベルにジャンプする).」
    
       (なお, _index がバッファ中の現在の終端位置を示す. 値が 0 であればもう空き領域はない)
      ---------------------------------------- -}

	  masm.ld_ptr(G2_thread, dirty_card_q_index_byte_offset, L0);
	
	  masm.br_on_reg_cond(Assembler::rc_z, /*annul*/false, Assembler::pn,
	                      L0, refill);

  {- -------------------------------------------
  (1) コード生成:
      「JavaThread::_dirty_card_queue フィールドの DirtyCardQueue オブジェクト内の
        _buf フィールドをロードする (なおこのロードは遅延スロットで実行される. ジャンプ先では使われないが害もない).」
      ---------------------------------------- -}

	  // If the branch is taken, no harm in executing this in the delay slot.
	  masm.delayed()->ld_ptr(G2_thread, dirty_card_q_buf_byte_offset, L1);

  {- -------------------------------------------
  (1) コード生成:
      「L0 レジスタの値を oopSize 分だけ小さくしておく.」      
      ---------------------------------------- -}

	  masm.sub(L0, oopSize, L0);
	
  {- -------------------------------------------
  (1) コード生成:
      「JavaThread::_dirty_card_queue フィールドの DirtyCardQueue オブジェクト内の
       _buf 内の "_index - oopSize" の位置に
       card table 中の該当する箇所のアドレスを書き込む.
       (より具体的に言うと, L0 と L1 を足したアドレスに O3 の値を書き込む)」
      ---------------------------------------- -}

	  masm.st_ptr(O3, L1, L0);  // [_buf + index] := I0

  {- -------------------------------------------
  (1) コード生成:
      「JavaThread::_dirty_card_queue フィールドの DirtyCardQueue オブジェクト内の
        _index フィールドに, 新しい値(oopSize 分だけ小さくした値)を書き込む.
        その後, リターンする.
        (正確には, 書き込み処理はリターンの遅延スロットで行われる)」
      ---------------------------------------- -}

	  // Use return-from-leaf
	  masm.retl();
	  masm.delayed()->st_ptr(L0, G2_thread, dirty_card_q_index_byte_offset);
	
  {- -------------------------------------------
  (1) (以降が refill ラベルに飛んだ場合の処理)
      ---------------------------------------- -}

	  masm.bind(refill);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  address handle_zero =
	    CAST_FROM_FN_PTR(address,
	                     &DirtyCardQueueSet::handle_zero_index_for_thread);

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

	  masm.mov(G1_scratch, L3);
	  masm.mov(G3_scratch, L5);

  {- -------------------------------------------
  (1) コード生成:
      「O0 レジスタの値が消えると困るので待避しておく.
        また, O7 レジスタの値も消えると困るので待避しておく」
      ---------------------------------------- -}

	  // We need the value of O3 above (for the write into the buffer), so we
	  // save and restore it.
	  masm.mov(O3, L6);
	  // Since the call will overwrite O7, we save and restore that, as well.
	  masm.mov(O7, L4);
	
  {- -------------------------------------------
  (1) コード生成:
      「MacroAssembler::call_VM_leaf() が生成するコードにより, 
        DirtyCardQueueSet::handle_zero_index_for_thread() を呼び出す.」
      ---------------------------------------- -}

	  masm.call_VM_leaf(L7_thread_cache, handle_zero, G2_thread);

  {- -------------------------------------------
  (1) コード生成:
      「待避していたレジスタを復帰し, restart ラベルに戻って処理をやり直す.」
      ---------------------------------------- -}

	  masm.mov(L3, G1_scratch);
	  masm.mov(L5, G3_scratch);
	  masm.mov(L6, O3);
	  masm.br(Assembler::always, /*annul*/false, Assembler::pt, restart);
	  masm.delayed()->mov(L4, O7);
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスと終端アドレスを
      対応する大域変数にセットする.
      ---------------------------------------- -}

	  dirty_card_log_enqueue = start;
	  dirty_card_log_enqueue_end = masm.pc();
	  // XXX Should have a guarantee here about not going off the end!
	  // Does it already do so?  Do an experiment...
	}
	
```


