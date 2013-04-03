---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateTable_sparc.cpp

### 名前(function name)
```
void TemplateTable::invokevirtual(int byte_no) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(vtos, vtos);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(byte_no == f2_byte, "use this argument");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Register Rscratch = G3_scratch;
	  Register Rtemp = G4_scratch;
	  Register Rret = Lscratch;
	  Register Rrecv = G5_method;
	  Label notFinal;
	
  {- -------------------------------------------
  (1) コード生成:
      「対応する methodOop の情報を CPCache から取得し, G5_method レジスタに設定する.
        (なお, この時点で resolve が終わってなければ resolve も行う)」
      ---------------------------------------- -}

	  load_invoke_cp_cache_entry(byte_no, G5_method, noreg, Rret, true, false, false);

  {- -------------------------------------------
  (1) コード生成:
      「SP の値を O5_savedSP レジスタに退避しておく」
      ---------------------------------------- -}

	  __ mov(SP, O5_savedSP); // record SP that we wanted the callee to restore
	
  {- -------------------------------------------
  (1) コード生成:
      「呼び出し対象のメソッドに final 修飾子が付いているかどうかを確認.
        付いていなければ notFinal ラベルに分岐.
        (なお, 後の receiver 取得処理で必要になるので, 
         オペランドスタック上に載っている引数の個数を取得する処理もついでに行っている)」
      ---------------------------------------- -}

	  // Check for vfinal
	  __ set((1 << ConstantPoolCacheEntry::vfinalMethod), G4_scratch);
	  __ btst(Rret, G4_scratch);
	  __ br(Assembler::zero, false, Assembler::pt, notFinal);
	  __ delayed()->and3(Rret, 0xFF, G4_scratch);      // gets number of parameters

  {- -------------------------------------------
  (1) (以下は, final 修飾子が付いていた場合のコード生成)
      ---------------------------------------- -}
	
    {- -------------------------------------------
  (1.1) コード生成:
        「このバイトコードを Bytecodes::_fast_invokevfinal に書き換えておく.
          (これにより次回以降の呼び出しが少し速くなる)」
        ---------------------------------------- -}

	  patch_bytecode(Bytecodes::_fast_invokevfinal, Rscratch, Rtemp);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「実際の呼び出し処理を行う」
        ---------------------------------------- -}

	  invokevfinal_helper(Rscratch, Rret);

  {- -------------------------------------------
  (1) (ここまでが, final 修飾子が付いていた場合)
      ---------------------------------------- -}
	
  {- -------------------------------------------
  (1) (以下は, final 修飾子が付いていなかった場合のコード生成)
      ---------------------------------------- -}

	  __ bind(notFinal);
	
    {- -------------------------------------------
  (1.1) コード生成: 
        「呼び出し先の methodOop のアドレスを Rscratch レジスタに入れる」
        ---------------------------------------- -}

	  __ mov(G5_method, Rscratch);  // better scratch register

    {- -------------------------------------------
  (1.1) コード生成: 
        「receiver のアドレスを O0 レジスタにロードする」
        ---------------------------------------- -}

	  __ load_receiver(G4_scratch, O0);  // gets receiverOop

    {- -------------------------------------------
  (1.1) コード生成: (verify)
        ---------------------------------------- -}

	  // receiver is in O0
	  __ verify_oop(O0);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「対応する return entry のアドレスを取得して Rret に入れる.
          (ここでは, 3byte先のバイトコードにdispatchする return entry を取得)」
        ---------------------------------------- -}

	  // get return address
	  AddressLiteral table(Interpreter::return_3_addrs_by_index_table());
	  __ set(table, Rtemp);
	  __ srl(Rret, ConstantPoolCacheEntry::tosBits, Rret);          // get return type
	  // Make sure we don't need to mask Rret for tosBits after the above shift
	  ConstantPoolCacheEntry::verify_tosBits();
	  __ sll(Rret,  LogBytesPerWord, Rret);
	  __ ld_ptr(Rtemp, Rret, Rret);         // get return address
	
    {- -------------------------------------------
  (1.1) コード生成:
        「receiver の klass フィールドが NULL でないことを確認した後, 
          クラスオブジェクトを Rrecv にロードする.
          NULL の場合は NullPointerException (See: [here](no30592Qc.html) for details)」
        ---------------------------------------- -}

	  // get receiver klass
	  __ null_check(O0, oopDesc::klass_offset_in_bytes());
	  __ load_klass(O0, Rrecv);

    {- -------------------------------------------
  (1.1) コード生成: (verify)
        ---------------------------------------- -}

	  __ verify_oop(Rrecv);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「method data pointer (mdp) の値を更新しておく」
        ---------------------------------------- -}

	  __ profile_virtual_call(Rrecv, O4);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「実際の呼び出し処理を行う」
        ---------------------------------------- -}

	  generate_vtable_call(Rrecv, Rscratch, Rret);

  {- -------------------------------------------
  (1) (ここまでが, final 修飾子が付いていなかった場合)
      ---------------------------------------- -}

	}
	
```


