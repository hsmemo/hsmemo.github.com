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
void TemplateTable::invokeinterface(int byte_no) {
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

	  assert(byte_no == f1_byte, "use this argument");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Register Rscratch = G4_scratch;
	  Register Rret = G3_scratch;
	  Register Rindex = Lscratch;
	  Register Rinterface = G1_scratch;
	  Register RklassOop = G5_method;
	  Register Rflags = O1;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(Rscratch, G5_method);
	
  {- -------------------------------------------
  (1) コード生成:
      「対応する methodOop の情報を CPCache から取得し, G5_method レジスタに設定する.
        (なお, この時点で resolve が終わってなければ resolve も行う)」
      ---------------------------------------- -}

	  load_invoke_cp_cache_entry(byte_no, Rinterface, Rindex, Rflags, /*virtual*/ false, false, false);

  {- -------------------------------------------
  (1) コード生成:
      「SP の値を O5_savedSP レジスタに退避しておく」
      ---------------------------------------- -}

	  __ mov(SP, O5_savedSP); // record SP that we wanted the callee to restore
	
  {- -------------------------------------------
  (1) コード生成:
      「receiver のオブジェクトを O0 レジスタにロード」
      ---------------------------------------- -}

	  // get receiver
	  __ and3(Rflags, 0xFF, Rscratch);       // gets number of parameters
	  __ load_receiver(Rscratch, O0);

  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_oop(O0);
	
  {- -------------------------------------------
  (1) コード生成:
      「対応する return entry のアドレスを取得して Rret に入れる.
       (ここでは, 5byte先のバイトコードにdispatchする return entry を取得)」
      ---------------------------------------- -}

	  __ mov(Rflags, Rret);
	
	  // get return address
	  AddressLiteral table(Interpreter::return_5_addrs_by_index_table());
	  __ set(table, Rscratch);
	  __ srl(Rret, ConstantPoolCacheEntry::tosBits, Rret);          // get return type
	  // Make sure we don't need to mask Rret for tosBits after the above shift
	  ConstantPoolCacheEntry::verify_tosBits();
	  __ sll(Rret,  LogBytesPerWord, Rret);
	  __ ld_ptr(Rscratch, Rret, Rret);      // get return address
	
  {- -------------------------------------------
  (1) コード生成:
      「receiver の klass フィールドが NULL でないことを確認した後, 
        クラスオブジェクトを Rrecv にロードする.
        NULL の場合は NullPointerException (See: [here](no30592Qc.html) for details)」
      ---------------------------------------- -}

	  // get receiver klass
	  __ null_check(O0, oopDesc::klass_offset_in_bytes());
	  __ load_klass(O0, RklassOop);

  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_oop(RklassOop);
	
  {- -------------------------------------------
  (1) コード生成:
      「仕様上は, invokeinterface で java.lang.Object のメソッドを呼び出すことが可能なので, 
        そのケースについて確認しておく.
        (なお, javac を使った場合には, そういったコードが出されることはない)
  
        このケースに該当する場合は, TemplateTable::invokeinterface_object_method() で
        生成するコードによって呼び出し処理を行う.」
  
      (該当する JavaVM 仕様の記述箇所は?? #TODO)
      ---------------------------------------- -}

	  // Special case of invokeinterface called for virtual method of
	  // java.lang.Object.  See cpCacheOop.cpp for details.
	  // This code isn't produced by javac, but could be produced by
	  // another compliant java compiler.
	  Label notMethod;
	  __ set((1 << ConstantPoolCacheEntry::methodInterface), Rscratch);
	  __ btst(Rflags, Rscratch);
	  __ br(Assembler::zero, false, Assembler::pt, notMethod);
	  __ delayed()->nop();
	
	  invokeinterface_object_method(RklassOop, Rinterface, Rret, Rflags);
	
	  __ bind(notMethod);
	
  {- -------------------------------------------
  (1) コード生成:
      「method data pointer (mdp) の値を更新しておく」
      ---------------------------------------- -}

	  __ profile_virtual_call(RklassOop, O4);
	
  {- -------------------------------------------
  (1) (以降で, 実際の呼び出し先を探す処理を行う)
      ---------------------------------------- -}

	  //
	  // find entry point to call
	  //
	
  {- -------------------------------------------
  (1) (以下の処理で, まず対応する itableOffsetEntry を探す.
       なお, ここで行っている内容は, instanceKlass::method_at_itable() の処理とほぼ同様.
       See: instanceKlass::method_at_itable())
      ---------------------------------------- -}

	  // compute start of first itableOffsetEntry (which is at end of vtable)

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	  const int base = instanceKlass::vtable_start_offset() * wordSize;
	  Label search;
	  Register Rtemp = Rflags;
	
    {- -------------------------------------------
  (1.1) コード生成:
        「クラスオブジェクト内にある itable の先頭アドレスを計算し, Rscratch に入れる」
  
         (なお, この処理は instanceKlass::start_of_itable() とほぼ同様.
          See: instanceKlass::start_of_itable())
        ---------------------------------------- -}

	  __ ld(RklassOop, instanceKlass::vtable_length_offset() * wordSize, Rtemp);
	  if (align_object_offset(1) > 1) {
	    __ round_to(Rtemp, align_object_offset(1));
	  }
	  __ sll(Rtemp, LogBytesPerWord, Rtemp);   // Rscratch *= 4;
	  if (Assembler::is_simm13(base)) {
	    __ add(Rtemp, base, Rtemp);
	  } else {
	    __ set(base, Rscratch);
	    __ add(Rscratch, Rtemp, Rtemp);
	  }
	  __ add(RklassOop, Rtemp, Rscratch);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「itable の中を先頭から順に調べていき, 
         _interface フィールドの値が目的のインターフェースに等しい要素を探す.
  
         なお, 探している途中で _interface フィールドが NULL の要素に出くわした場合は
         IncompatibleClassChangeError を出す.」
        ---------------------------------------- -}

	  __ bind(search);
	
    {- -------------------------------------------
  (1.1) (処理対象の itableOffsetEntry から _interface フィールドをロードする)
        ---------------------------------------- -}

	  __ ld_ptr(Rscratch, itableOffsetEntry::interface_offset_in_bytes(), Rtemp);

    {- -------------------------------------------
  (1.1) (ロード結果が NULL だったら, IncompatibleClassChangeError)
        ---------------------------------------- -}

	  {
	    Label ok;
	
	    // Check that entry is non-null.  Null entries are probably a bytecode
	    // problem.  If the interface isn't implemented by the receiver class,
	    // the VM should throw IncompatibleClassChangeError.  linkResolver checks
	    // this too but that's only if the entry isn't already resolved, so we
	    // need to check again.
	    __ br_notnull( Rtemp, false, Assembler::pt, ok);
	    __ delayed()->nop();
	    call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::throw_IncompatibleClassChangeError));
	    __ should_not_reach_here();
	    __ bind(ok);
	    __ verify_oop(Rtemp);
	  }
	
	  __ verify_oop(Rinterface);
	
    {- -------------------------------------------
  (1.1) (ロード結果が探しているインターフェースと等しければここで終了.
         違っていれば, 処理対象を次の要素に移した後, search ラベルまで戻ってやり直し)
        ---------------------------------------- -}

	  __ cmp(Rinterface, Rtemp);
	  __ brx(Assembler::notEqual, true, Assembler::pn, search);
	  __ delayed()->add(Rscratch, itableOffsetEntry::size() * wordSize, Rscratch);

  {- -------------------------------------------
  (1) (対応する itableOffsetEntry を探す処理はここまで)
      ---------------------------------------- -}
	
  {- -------------------------------------------
  (1) コード生成:
      「探し当てた itableOffsetEntry 内の offset 値から適切な vtable の位置を求め, 
        さらに vtable のアドレスに index を足すことで呼び出し先の methodOop を取得する.
        取得した結果は, G5_method レジスタに格納.」
      ---------------------------------------- -}

	  // entry found and Rscratch points to it
	  __ ld(Rscratch, itableOffsetEntry::offset_offset_in_bytes(), Rscratch);
	
	  assert(itableMethodEntry::method_offset_in_bytes() == 0, "adjust instruction below");
	  __ sll(Rindex, exact_log2(itableMethodEntry::size() * wordSize), Rindex);       // Rindex *= 8;
	  __ add(Rscratch, Rindex, Rscratch);
	  __ ld_ptr(RklassOop, Rscratch, G5_method);
	
  {- -------------------------------------------
  (1) コード生成:
      「もし取得した methodOop が NULL だったら, AbstractMethodError を出す.」
      ---------------------------------------- -}

	  // Check for abstract method error.
	  {
	    Label ok;
	    __ tst(G5_method);
	    __ brx(Assembler::notZero, false, Assembler::pt, ok);
	    __ delayed()->nop();
	    call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::throw_AbstractMethodError));
	    __ should_not_reach_here();
	    __ bind(ok);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Register Rcall = Rinterface;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(Rcall, G5_method, Gargs, Rret);
	
  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_oop(G5_method);

  {- -------------------------------------------
  (1) コード生成:
      「実際の呼び出し処理を行う」
      ---------------------------------------- -}

	  __ call_from_interpreter(Rcall, Gargs, Rret);
	
	}
	
```


