---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interp_masm_x86_64.cpp
### 説明(description)

```
// Lock object
//
// Args:
//      c_rarg1: BasicObjectLock to be used for locking
//
// Kills:
//      rax
//      c_rarg0, c_rarg1, c_rarg2, c_rarg3, .. (param regs)
//      rscratch1, rscratch2 (scratch regs)
```

### 名前(function name)
```
void InterpreterMacroAssembler::lock_object(Register lock_reg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数では, UseHeavyMonitors オプションの値に応じて 2通りのコードを生成)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(lock_reg == c_rarg1, "The argument is only for looks. It must be c_rarg1");
	
  {- -------------------------------------------
  (1) コード生成: (こちらは, UseHeavyMonitors オプションが指定されている場合に生成するコード)            
      「(この場合は軽量ロック処理(fast-path の処理)は行わないので)
       InterpreterRuntime::monitorenter() を呼び出して slow-path のコードにフォールバック」
      ---------------------------------------- -}

	  if (UseHeavyMonitors) {
	    call_VM(noreg,
	            CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorenter),
	            lock_reg);

  {- -------------------------------------------
  (1) コード生成: (こちらは, UseHeavyMonitors オプションが指定されていない場合に生成するコード)
      (この場合は以下で示すように, 
       まず fast-path の処理を行い, それがダメな場合に
       InterpreterRuntime::monitorenter() にフォールバックするコードを生成)
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    Label done;
	
	    const Register swap_reg = rax; // Must use rax for cmpxchg instruction
	    const Register obj_reg = c_rarg3; // Will contain the oop
	
	    const int obj_offset = BasicObjectLock::obj_offset_in_bytes();
	    const int lock_offset = BasicObjectLock::lock_offset_in_bytes ();
	    const int mark_offset = lock_offset +
	                            BasicLock::displaced_header_offset_in_bytes();
	
	    Label slow_case;
	
    {- -------------------------------------------
  (1.1) コード生成:
        「ロック対象のオブジェクトの mark field を取得」
        ---------------------------------------- -}

	    // Load object pointer into obj_reg %c_rarg3
	    movptr(obj_reg, Address(lock_reg, obj_offset));
	
    {- -------------------------------------------
  (1.1) コード生成: (ただし, UseBiasedLocking オプションが指定されていない場合には, 不要なので生成しない)
        「MacroAssembler::biased_locking_enter() が生成するコードにより, biased locking 処理を行う」  
        (なおこの場合, ロック確保が成功すれば done ラベルに分岐する. 
        失敗時にはこのままフォールスルーするか slow_case ラベルに分岐する)
        ---------------------------------------- -}

	    if (UseBiasedLocking) {
	      biased_locking_enter(lock_reg, obj_reg, swap_reg, rscratch1, false, done, &slow_case);
	    }
	
    {- -------------------------------------------
  (1.1) コード生成:
        「(mark field | 1) という値を BasicLock に書き込む (mark field の退避処理)」
        ---------------------------------------- -}

	    // Load immediate 1 into swap_reg %rax
	    movl(swap_reg, 1);
	
	    // Load (object->mark() | 1) into swap_reg %rax
	    orptr(swap_reg, Address(obj_reg, 0));
	
	    // Save (object->mark() | 1) into BasicLock's displaced header
	    movptr(Address(lock_reg, mark_offset), swap_reg);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「まずは neutral だと想定して mark field を CAS で置き換える. 
         成功すれば (本当に neutral だったということで) ここでロック処理は終了(done ラベルに分岐する)」
        ---------------------------------------- -}

	    assert(lock_offset == 0,
	           "displached header must be first word in BasicObjectLock");
	
	    if (os::is_MP()) lock();
	    cmpxchgptr(lock_reg, Address(obj_reg, 0));
	    if (PrintBiasedLockingStatistics) {
	      cond_inc32(Assembler::zero,
	                 ExternalAddress((address) BiasedLocking::fast_path_entry_count_addr()));
	    }
	    jcc(Assembler::zero, done);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「CAS が成功しなかった場合は, 自分がロック済み (かつ stack-locked) かどうかを確認する.
        そうだった場合にはここでロック処理は終了(done ラベルに分岐する).
        (この場合 BasicLock には NULL を入れておく)」
        ---------------------------------------- -}

	    // Test if the oopMark is an obvious stack pointer, i.e.,
	    //  1) (mark & 7) == 0, and
	    //  2) rsp <= mark < mark + os::pagesize()
	    //
	    // These 3 tests can be done by evaluating the following
	    // expression: ((mark - rsp) & (7 - os::vm_page_size())),
	    // assuming both stack pointer and pagesize have their
	    // least significant 3 bits clear.
	    // NOTE: the oopMark is in swap_reg %rax as the result of cmpxchg
	    subptr(swap_reg, rsp);
	    andptr(swap_reg, 7 - os::vm_page_size());
	
	    // Save the test result, for recursive case, the result is zero
	    movptr(Address(lock_reg, mark_offset), swap_reg);
	
	    if (PrintBiasedLockingStatistics) {
	      cond_inc32(Assembler::zero,
	                 ExternalAddress((address) BiasedLocking::fast_path_entry_count_addr()));
	    }
	    jcc(Assembler::zero, done);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「それも成功しなかった場合は, 
         InterpreterRuntime::monitorenter() を呼び出して slow-path のコードにフォールバック」
    
        (なお, ここが slow_case ラベルの位置)
        ---------------------------------------- -}

	    bind(slow_case);
	
	    // Call the runtime routine for slow case
	    call_VM(noreg,
	            CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorenter),
	            lock_reg);
	
    {- -------------------------------------------
  (1.1) (ここが done ラベルの位置)
        ---------------------------------------- -}

	    bind(done);
	  }
	
```


