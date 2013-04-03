---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateTable_x86_64.cpp

### 名前(function name)
```
void TemplateTable::_new() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(vtos, atos);

  {- -------------------------------------------
  (1) コード生成:
      「constant pool の index を rdx に取得しておく」
      ---------------------------------------- -}

	  __ get_unsigned_2_byte_index_at_bcp(rdx, 1);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label slow_case;
	  Label done;
	  Label initialize_header;
	  Label initialize_object; // including clearing the fields
	  Label allocate_shared;
	
  {- -------------------------------------------
  (1) コード生成:
      「現在実行中のメソッドに対応する constant pool を取得し, 
       あわせて new 対象のクラスオブジェクトに対応する tag を取得する.」
      ---------------------------------------- -}

	  __ get_cpool_and_tags(rsi, rax);

  {- -------------------------------------------
  (1) コード生成:
      「new 対象のクラスが resolve 済みかどうかを確認する.
       もし, resolve 済みでなければ (= constant pool の該当エントリが JVM_CONSTANT_Class でなければ (See: [here](no2935KOa.html) for details)), 
       fast-path での処理はできないので, slow_case ラベルにジャンプする.」
      ---------------------------------------- -}

	  // Make sure the class we're about to instantiate has been resolved.
	  // This is done before loading instanceKlass to be consistent with the order
	  // how Constant Pool is updated (see constantPoolOopDesc::klass_at_put)
	  const int tags_offset = typeArrayOopDesc::header_size(T_BYTE) * wordSize;
	  __ cmpb(Address(rax, rdx, Address::times_1, tags_offset),
	          JVM_CONSTANT_Class);
	  __ jcc(Assembler::notEqual, slow_case);
	
  {- -------------------------------------------
  (1) コード生成:
      「new 対象のクラスオブジェクトを取得する (= constant pool から対応するエントリを取得する).」
      ---------------------------------------- -}

	  // get instanceKlass
	  __ movptr(rsi, Address(rsi, rdx,
	            Address::times_8, sizeof(constantPoolOopDesc)));
	
  {- -------------------------------------------
  (1) コード生成:
      「もし, クラスの初期化が完全に終わっていない場合には 
       (= ...(#TODO) が instanceKlass::fully_initialized ではない場合には), 
       fast-path での処理はできないので, slow_case ラベルにジャンプする.」
      ---------------------------------------- -}

	  // make sure klass is initialized & doesn't have finalizer
	  // make sure klass is fully initialized
	  __ cmpl(Address(rsi,
	                  instanceKlass::init_state_offset_in_bytes() +
	                  sizeof(oopDesc)),
	          instanceKlass::fully_initialized);
	  __ jcc(Assembler::notEqual, slow_case);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // get instance_size in instanceKlass (scaled to a count of bytes)
	  __ movl(rdx,
	          Address(rsi,
	                  Klass::layout_helper_offset_in_bytes() + sizeof(oopDesc)));

  {- -------------------------------------------
  (1) コード生成:
      「もし, 対象のクラスが (abstract や interface 等) new できないものである場合には
       (= Klass::_lh_instance_slow_path_bit が立っている場合には), 
       fast-path での処理はできないので, slow_case ラベルにジャンプする.」
      ---------------------------------------- -}

	  // test to see if it has a finalizer or is malformed in some way
	  __ testl(rdx, Klass::_lh_instance_slow_path_bit);
	  __ jcc(Assembler::notZero, slow_case);
	
  {- -------------------------------------------
  (1) (これ以降が fast-path での確保処理.
       fast-path では以下のようにして確保を試みることになる. 
       これらが失敗した場合は, slow-path での確保処理にフォールスルーする.
       (1) まず TLAB 内に確保を試みる. 
       (2) TLAB での確保が失敗したが, TLAB にまだ充分空きがある場合には, 
           現在の TLAB を捨てるのはもったいないので shared Eden 内に直接確保を試みる.)
      ---------------------------------------- -}

	  // Allocate the instance
	  // 1) Try to allocate in the TLAB
	  // 2) if fail and the object is large allocate in the shared Eden
	  // 3) if the above fails (or is not applicable), go to a slow case
	  // (creates a new TLAB, etc.)
	
  {- -------------------------------------------
  (1) (変数宣言など）
      (allow_shared_alloc は, shared eden からの確保を試みるかどうかを示す.
       基本的には Universe::heap()->supports_inline_contig_alloc() が true であれば試みるが, 
       CMSIncrementalMode の場合にだけは試みない)
      ---------------------------------------- -}

	  const bool allow_shared_alloc =
	    Universe::heap()->supports_inline_contig_alloc() && !CMSIncrementalMode;
	
  {- -------------------------------------------
  (1) コード生成: (ただし, UseTLAB オプションが指定されている場合にしか行わない)
      「まず以下の if ブロックが生成するコード内で TLAB からの確保を試みる.
       確保が成功すれば, initialize_header ラベルまたは initialize_object ラベルに分岐する.
        (なお, initialize_header ラベルと initialize_object ラベルの違いは, 
        確保時にオブジェクトのフィールド部分をゼロクリアするか否か)
  
       確保が失敗した場合は, shared eden からの確保処理にフォールスルーするか, 
       もしくは snow_case での確保処理を行う.」
      ---------------------------------------- -}

	  if (UseTLAB) {

    {- -------------------------------------------
  (1.1) コード生成:
        「カレントスレッドの TLAB について, 現在使用量 (top 値) と TLAB の終端値 (end 値) を取得し, 
         「現在使用量+確保サイズ」を計算する.」
        ---------------------------------------- -}

	    __ movptr(rax, Address(r15_thread, in_bytes(JavaThread::tlab_top_offset())));
	    __ lea(rbx, Address(rax, rdx, Address::times_1));
	    __ cmpptr(rbx, Address(r15_thread, in_bytes(JavaThread::tlab_end_offset())));

    {- -------------------------------------------
  (1.1) コード生成:
        「TLAB サイズより「現在使用量+確保サイズ」が大きい場合には
         allocate_shared ラベルもしくは slow_case ラベルに分岐する」
        ---------------------------------------- -}

	    __ jcc(Assembler::above, allow_shared_alloc ? allocate_shared : slow_case);

    {- -------------------------------------------
  (1.1) コード生成:
        「(TLAB サイズが「現在使用量+確保サイズ」より小さい場合には)
          TLAB の現在使用量 (top 値) を新しい値に更新した後, 
          ZeroTLAB オプションの価に応じて initialize_header または initialize_object ラベルにジャンプする.」
        ---------------------------------------- -}

	    __ movptr(Address(r15_thread, in_bytes(JavaThread::tlab_top_offset())), rbx);
	    if (ZeroTLAB) {
	      // the fields have been already cleared
	      __ jmp(initialize_header);
	    } else {
	      // initialize both the header and fields
	      __ jmp(initialize_object);
	    }
	  }
	
  {- -------------------------------------------
  (1) (以下は, shared eden からの確保を試みるパス. 
       当然だが, このコードは shared eden が利用可能な場合にしか生成されない.
  
       なお, このパスが実行されるのは, 
       UseTLAB オプションが指定されていない場合, もしくは 上記の TLAB からの確保処理が失敗した場合のみ)
      ---------------------------------------- -}

	  // Allocation in the shared Eden, if allowed.
	  //
	  // rdx: instance size in bytes

    {- -------------------------------------------
  (1.1) コード生成: (allow_shared_alloc が true の場合にのみ生成)
        「shared eden からの確保処理を行う.
  
         * 確保が成功すれば, initialize_object ラベルにフォールスルー
           (<= なお, TLAB からの確保時と異なり ZeroTLAB オプションの値によって飛び先が変わることはない. まぁ TLAB ではないので当たり前か)
         * 確保が失敗すれば, slow_case での処理に分岐する.」 
        ---------------------------------------- -}

	  if (allow_shared_alloc) {
	    __ bind(allocate_shared);
	
	    ExternalAddress top((address)Universe::heap()->top_addr());
	    ExternalAddress end((address)Universe::heap()->end_addr());
	
	    const Register RtopAddr = rscratch1;
	    const Register RendAddr = rscratch2;
	
      {- -------------------------------------------
  (1.1.1) コード生成:
          「ヒープの eden 領域について, 現在使用量 (top 値) を取得.」
          ---------------------------------------- -}

	    __ lea(RtopAddr, top);
	    __ lea(RendAddr, end);
	    __ movptr(rax, Address(RtopAddr, 0));
	
      {- -------------------------------------------
  (1.1.1) コード生成:
          「cas で Universe::heap()->top を書き換えることでメモリを確保する. cas が失敗した場合は処理を繰り返す.
            なお, 空き容量が確保量よりも小さい場合は, 諦めて slow_case にジャンプする.」
          ---------------------------------------- -}

	    // For retries rax gets set by cmpxchgq
	    Label retry;
	    __ bind(retry);
	    __ lea(rbx, Address(rax, rdx, Address::times_1));
	    __ cmpptr(rbx, Address(RendAddr, 0));
	    __ jcc(Assembler::above, slow_case);
	
	    // Compare rax with the top addr, and if still equal, store the new
	    // top addr in rbx at the address of the top addr pointer. Sets ZF if was
	    // equal, and clears it otherwise. Use lock prefix for atomicity on MPs.
	    //
	    // rax: object begin
	    // rbx: object end
	    // rdx: instance size in bytes
	    if (os::is_MP()) {
	      __ lock();
	    }
	    __ cmpxchgptr(rbx, Address(RtopAddr, 0));
	
	    // if someone beat us on the allocation, try again, otherwise continue
	    __ jcc(Assembler::notEqual, retry);
	
      {- -------------------------------------------
  (1.1.1) コード生成:
          「cas が成功していれば, MacroAssembler::incr_allocated_bytes() で統計情報を更新しておく. 
            (See: [here](no2114Q4Z.html) for details)」
          ---------------------------------------- -}

	    __ incr_allocated_bytes(r15_thread, rdx, 0);
	  }
	
  {- -------------------------------------------
  (1) (以下は fast-path が成功し, かつ...#TODO 場合の処理パス.
       より具体的には, initialize_object ラベルの bind 箇所)
  
      (なお, allow_shared_alloc 変数の定義より, 
      allow_shared_alloc が true の場合には Universe::heap()->supports_inline_contig_alloc() も true である.
      このため, 上の shared eden の確保処理パスが作られる場合には, この initialize_object ラベルの処理パスも作られる.
      <= というか, 作られない場合には shared eden の処理成功時のフォールスルー先が snow_case になってしまうので, 
         fast_path の意味が無い)
      ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) コード生成: (ただし, UseTLAB オプションが指定されている場合, または)
        「確保したオブジェクトにフィールドがなければ, 単に initialize_header ラベルにフォールスルーするだけ.
         フィールドがあれば, フィールドを全て 0 クリアした後, initialize_header ラベルにフォールスルーする」
        ---------------------------------------- -}

	  if (UseTLAB || Universe::heap()->supports_inline_contig_alloc()) {
	    // The object is initialized before the header.  If the object size is
	    // zero, go directly to the header initialization.
	    __ bind(initialize_object);
	    __ decrementl(rdx, sizeof(oopDesc));
	    __ jcc(Assembler::zero, initialize_header);
	
	    // Initialize object fields
	    __ xorl(rcx, rcx); // use zero reg to clear memory (shorter code)
	    __ shrl(rdx, LogBytesPerLong);  // divide by oopSize to simplify the loop
	    {
	      Label loop;
	      __ bind(loop);
	      __ movq(Address(rax, rdx, Address::times_8,
	                      sizeof(oopDesc) - oopSize),
	              rcx);
	      __ decrementl(rdx);
	      __ jcc(Assembler::notZero, loop);
	    }
	
  {- -------------------------------------------
  (1) (以下は fast-path が成功した場合の処理パス.
       より具体的には, initialize_header ラベルの bind 箇所)
      ---------------------------------------- -}

	    // initialize object header only.

    {- -------------------------------------------
  (1.1) コード生成:
        「確保したオブジェクトの mark フィールド, 及び klass フィールドを初期化する.
          (なお, (DTrace のフック点) でもある).
  
         その後 done ラベルへとフォールスルーする.」
        ---------------------------------------- -}

	    __ bind(initialize_header);
	    if (UseBiasedLocking) {
	      __ movptr(rscratch1, Address(rsi, Klass::prototype_header_offset_in_bytes() + klassOopDesc::klass_part_offset_in_bytes()));
	      __ movptr(Address(rax, oopDesc::mark_offset_in_bytes()), rscratch1);
	    } else {
	      __ movptr(Address(rax, oopDesc::mark_offset_in_bytes()),
	               (intptr_t) markOopDesc::prototype()); // header (address 0x1)
	    }
	    __ xorl(rcx, rcx); // use zero reg to clear memory (shorter code)
	    __ store_klass_gap(rax, rcx);  // zero klass gap for compressed oops
	    __ store_klass(rax, rsi);      // store klass last
	
	    {
	      SkipIfEqual skip(_masm, &DTraceAllocProbes, false);
	      // Trigger dtrace event for fastpath
	      __ push(atos); // save the return value
	      __ call_VM_leaf(
	           CAST_FROM_FN_PTR(address, SharedRuntime::dtrace_object_alloc), rax);
	      __ pop(atos); // restore the return value
	
	    }
	    __ jmp(done);
	  }
	
	
  {- -------------------------------------------
  (1) (以下は fast-path が失敗した場合の処理パス. 
       つまりは, slow-path の処理パス. より具体的には, slow_case ラベルの bind 箇所)
      ---------------------------------------- -}

	  // slow case

    {- -------------------------------------------
  (1.1) コード生成:
        「InterpreterRuntime::_new() を呼び出すことで確保処理を行う. その後 done ラベルまでジャンプする.」
        ---------------------------------------- -}

	  __ bind(slow_case);
	  __ get_constant_pool(c_rarg1);
	  __ get_unsigned_2_byte_index_at_bcp(c_rarg2, 1);
	  call_VM(rax, CAST_FROM_FN_PTR(address, InterpreterRuntime::_new), c_rarg1, c_rarg2);
	  __ verify_oop(rax);
	
  {- -------------------------------------------
  (1) (以下が, 確保処理が終わった後のジャンプ先.
       より具体的には, done ラベルの bind 箇所)
      ---------------------------------------- -}

	  // continue
	  __ bind(done);
	}
	
```


