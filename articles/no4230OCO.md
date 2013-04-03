---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interp_masm_sparc.cpp
### 説明(description)

```
// Lock object
//
// Argument - lock_reg points to the BasicObjectLock to be used for locking,
//            it must be initialized with the object to lock
```

### 名前(function name)
```
void InterpreterMacroAssembler::lock_object(Register lock_reg, Register Object) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数では, UseHeavyMonitors オプションの値に応じて 2通りのコードを生成)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) コード生成: (こちらは, UseHeavyMonitors オプションが指定されている場合に生成するコード)            
      「(この場合は軽量ロック処理(fast-path の処理)は行わないので)
       InterpreterRuntime::monitorenter() を呼び出して slow-path のコードにフォールバック」
      ---------------------------------------- -}

	  if (UseHeavyMonitors) {
	    call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorenter), lock_reg);
	  }

  {- -------------------------------------------
  (1) コード生成: (こちらは, UseHeavyMonitors オプションが指定されていない場合に生成するコード)
      (この場合は以下で示すように, 
       まず fast-path の処理を行い, それがダメな場合に
       InterpreterRuntime::monitorenter() にフォールバックするコードを生成)
      ---------------------------------------- -}

	  else {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    Register obj_reg = Object;
	    Register mark_reg = G4_scratch;
	    Register temp_reg = G1_scratch;
	    Address  lock_addr(lock_reg, BasicObjectLock::lock_offset_in_bytes());
	    Address  mark_addr(obj_reg, oopDesc::mark_offset_in_bytes());
	    Label    done;
	
	    Label slow_case;
	
	    assert_different_registers(lock_reg, obj_reg, mark_reg, temp_reg);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「ロック対象のオブジェクトの mark field を取得」
        ---------------------------------------- -}

	    // load markOop from object into mark_reg
	    ld_ptr(mark_addr, mark_reg);
	
    {- -------------------------------------------
  (1.1) コード生成: (ただし, UseBiasedLocking オプションが指定されていない場合には, 不要なので生成しない)
        「MacroAssembler::biased_locking_enter() が生成するコードにより, biased locking 処理を行う」
        (なおこの場合, ロック確保が成功すれば done ラベルに分岐する. 
        失敗時にはこのままフォールスルーするか slow_case ラベルに分岐する)
        ---------------------------------------- -}

	    if (UseBiasedLocking) {
	      biased_locking_enter(obj_reg, mark_reg, temp_reg, done, &slow_case);
	    }
	
    {- -------------------------------------------
  (1.1) コード生成:
        「lock_reg の値を潰されるとまずいので, 別のレジスタに値をコピーしておく (CAS すると値が潰されてしまうので)」
        ---------------------------------------- -}

	    // get the address of basicLock on stack that will be stored in the object
	    // we need a temporary register here as we do not want to clobber lock_reg
	    // (cas clobbers the destination register)
	    mov(lock_reg, temp_reg);

    {- -------------------------------------------
  (1.1) コード生成:
        「(mark field | UNLOCK_VALUE) という値を BasicLock に書き込む (mark field の退避処理)」
        ---------------------------------------- -}

	    // set mark reg to be (markOop of object | UNLOCK_VALUE)
	    or3(mark_reg, markOopDesc::unlocked_value, mark_reg);
	    // initialize the box  (Must happen before we update the object mark!)
	    st_ptr(mark_reg, lock_addr, BasicLock::displaced_header_offset_in_bytes());

    {- -------------------------------------------
  (1.1) コード生成:
        「まずは neutral だと想定して mark field を CAS で置き換える. 
         成功すれば (本当に neutral だったということで) ここでロック処理は終了(done ラベルに分岐する)」
        ---------------------------------------- -}

	    // compare and exchange object_addr, markOop | 1, stack address of basicLock
	    assert(mark_addr.disp() == 0, "cas must take a zero displacement");
	    casx_under_lock(mark_addr.base(), mark_reg, temp_reg,
	      (address)StubRoutines::Sparc::atomic_memory_operation_lock_addr());
	
	    // if the compare and exchange succeeded we are done (we saw an unlocked object)
	    cmp(mark_reg, temp_reg);
	    brx(Assembler::equal, true, Assembler::pt, done);
	    delayed()->nop();
	
    {- -------------------------------------------
  (1.1) コード生成:
        「CAS が成功しなかった場合は, 自分がロック済み (かつ stack-locked) かどうかを確認する.
        そうだった場合にはここでロック処理は終了(done ラベルに分岐する).
        (この場合 BasicLock には NULL を入れておく)」
        ---------------------------------------- -}

	    // We did not see an unlocked object so try the fast recursive case
	
	    // Check if owner is self by comparing the value in the markOop of object
	    // with the stack pointer
	    sub(temp_reg, SP, temp_reg);
	#ifdef _LP64
	    sub(temp_reg, STACK_BIAS, temp_reg);
	#endif
	    assert(os::vm_page_size() > 0xfff, "page size too small - change the constant");
	
	    // Composite "andcc" test:
	    // (a) %sp -vs- markword proximity check, and,
	    // (b) verify mark word LSBs == 0 (Stack-locked).
	    //
	    // FFFFF003/FFFFFFFFFFFF003 is (markOopDesc::lock_mask_in_place | -os::vm_page_size())
	    // Note that the page size used for %sp proximity testing is arbitrary and is
	    // unrelated to the actual MMU page size.  We use a 'logical' page size of
	    // 4096 bytes.   F..FFF003 is designed to fit conveniently in the SIMM13 immediate
	    // field of the andcc instruction.
	    andcc (temp_reg, 0xFFFFF003, G0) ;
	
	    // if condition is true we are done and hence we can store 0 in the displaced
	    // header indicating it is a recursive lock and be done
	    brx(Assembler::zero, true, Assembler::pt, done);
	    delayed()->st_ptr(G0, lock_addr, BasicLock::displaced_header_offset_in_bytes());
	
    {- -------------------------------------------
  (1.1) コード生成:
        「それも成功しなかった場合は, 
         InterpreterRuntime::monitorenter() を呼び出して slow-path のコードにフォールバック」
    
        (なお, ここが slow_case ラベルの位置)
        ---------------------------------------- -}

	    // none of the above fast optimizations worked so we have to get into the
	    // slow case of monitor enter
	    bind(slow_case);
	    call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorenter), lock_reg);
	
    {- -------------------------------------------
  (1.1) (ここが done ラベルの位置)
        ---------------------------------------- -}

	    bind(done);
	  }
	
```


