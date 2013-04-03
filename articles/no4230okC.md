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
// Unlocks an object. Used in monitorexit bytecode and
// remove_activation.  Throws an IllegalMonitorException if object is
// not locked by current thread.
//
// Args:
//      c_rarg1: BasicObjectLock for lock
//
// Kills:
//      rax
//      c_rarg0, c_rarg1, c_rarg2, c_rarg3, ... (param regs)
//      rscratch1, rscratch2 (scratch regs)
```

### 名前(function name)
```
void InterpreterMacroAssembler::unlock_object(Register lock_reg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数では, UseHeavyMonitors オプションの値に応じて 2通りのコードを生成)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(lock_reg == c_rarg1, "The argument is only for looks. It must be rarg1");
	
  {- -------------------------------------------
  (1) コード生成: (こちらは, UseHeavyMonitors オプションが指定されている場合に生成するコード)            
      「(この場合は軽量ロック処理(fast-path の処理)は行わないので)
       InterpreterRuntime::monitorexit() を呼び出して slow-path のコードにフォールバック」
      ---------------------------------------- -}

	  if (UseHeavyMonitors) {
	    call_VM(noreg,
	            CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorexit),
	            lock_reg);

  {- -------------------------------------------
  (1) コード生成: (こちらは, UseHeavyMonitors オプションが指定されていない場合に生成するコード)
      (この場合は以下で示すように, 
       まず fast-path の処理を行い, それがダメな場合に
       InterpreterRuntime::monitorexit() にフォールバックするコードを生成)
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    Label done;
	
	    const Register swap_reg   = rax;  // Must use rax for cmpxchg instruction
	    const Register header_reg = c_rarg2;  // Will contain the old oopMark
	    const Register obj_reg    = c_rarg3;  // Will contain the oop
	
    {- -------------------------------------------
  (1.1) コード生成:
        「以下の処理の間, r13(bcp) を一時的に退避しておく」
        ---------------------------------------- -}

	    save_bcp(); // Save in case of exception
	
    {- -------------------------------------------
  (1.1) コード生成:
        「BasicObjectLock から oop と BasicLock を取り出し, BasicObjectLock 自体は開放する」
        ---------------------------------------- -}

	    // Convert from BasicObjectLock structure to object and BasicLock
	    // structure Store the BasicLock address into %rax
	    lea(swap_reg, Address(lock_reg, BasicObjectLock::lock_offset_in_bytes()));
	
	    // Load oop into obj_reg(%c_rarg3)
	    movptr(obj_reg, Address(lock_reg, BasicObjectLock::obj_offset_in_bytes()));
	
	    // Free entry
	    movptr(Address(lock_reg, BasicObjectLock::obj_offset_in_bytes()), (int32_t)NULL_WORD);
	
    {- -------------------------------------------
  (1.1) コード生成: (ただし, UseBiasedLocking オプションが指定されていない場合には, 不要なので生成しない)
        「MacroAssembler::biased_locking_exit() が生成するコードで biased locking の処理を行う.
         ロックの解放に成功すれば, ここで終了 (最後尾にある done までジャンプするコードが生成される)」
        ---------------------------------------- -}

	    if (UseBiasedLocking) {
	      biased_locking_exit(obj_reg, header_reg, done);
	    }
	
    {- -------------------------------------------
  (1.1) コード生成:
        「もし BasicObjectLock の _lock フィールドが NULL であれば 
          (= これは自分が再帰的にロックを取得していたケース), 
          ここで終了 (最後尾にある done までジャンプ)」
        ---------------------------------------- -}

	    // Load the old header from BasicLock structure
	    movptr(header_reg, Address(swap_reg,
	                               BasicLock::displaced_header_offset_in_bytes()));
	
	    // Test for recursion
	    testptr(header_reg, header_reg);
	
	    // zero for recursive case
	    jcc(Assembler::zero, done);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「もし軽量ロック(stack-locked)であれば, 待避していた mark フィールドの中身を CAS で書き戻す. 
          成功したら終了 (done までジャンプ)
  
          (正確に言うと, 自分が書き換えたときのままだと想定していきなり CAS する. 
          成功してたら自分が書き換えたときのままだった(= stack-lockedだった)ということになる)」
        ---------------------------------------- -}

	    // Atomic swap back the old header
	    if (os::is_MP()) lock();
	    cmpxchgptr(header_reg, Address(obj_reg, 0));
	
	    // zero for recursive case
	    jcc(Assembler::zero, done);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「CAS が失敗した場合には, 
        InterpreterRuntime::monitorexit() を呼び出して slow-path のコードにフォールバック」
        ---------------------------------------- -}

	    // Call the runtime routine for slow case.
	    movptr(Address(lock_reg, BasicObjectLock::obj_offset_in_bytes()),
	         obj_reg); // restore obj
	    call_VM(noreg,
	            CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorexit),
	            lock_reg);
	
    {- -------------------------------------------
  (1.1) (ここが done ラベルの位置)
        ---------------------------------------- -}

	    bind(done);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「r13(bcp) の値を復帰させる」
        ---------------------------------------- -}

	    restore_bcp();
	  }
	}
	
```


