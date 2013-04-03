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
// Unlocks an object. Used in monitorexit bytecode and remove_activation.
//
// Argument - lock_reg points to the BasicObjectLock for lock
// Throw IllegalMonitorException if object is not locked by current thread
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
  (1) コード生成: (こちらは, UseHeavyMonitors オプションが指定されている場合に生成するコード)            
      「(この場合は軽量ロック処理(fast-path の処理)は行わないので)
       InterpreterRuntime::monitorexit() を呼び出して slow-path のコードにフォールバック」
      ---------------------------------------- -}

	  if (UseHeavyMonitors) {
	    call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorexit), lock_reg);

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

	    Register obj_reg = G3_scratch;
	    Register mark_reg = G4_scratch;
	    Register displaced_header_reg = G1_scratch;
	    Address  lockobj_addr(lock_reg, BasicObjectLock::obj_offset_in_bytes());
	    Address  mark_addr(obj_reg, oopDesc::mark_offset_in_bytes());
	    Label    done;
	
    {- -------------------------------------------
  (1.1) コード生成: (ただし, UseBiasedLocking オプションが指定されていない場合には, 不要なので生成しない)
        「MacroAssembler::biased_locking_exit() が生成するコードで biased locking の処理を行う.
         ロックの解放に成功すれば, ここで終了 (最後尾にある done までジャンプするコードが生成される)
  
         (なお, 対応する BasicObjectLock を開放するための st_ptr は, done までジャンプする際の遅延スロットで実行される)」
        ---------------------------------------- -}

	    if (UseBiasedLocking) {
	      // load the object out of the BasicObjectLock
	      ld_ptr(lockobj_addr, obj_reg);
	      biased_locking_exit(mark_addr, mark_reg, done, true);
	      st_ptr(G0, lockobj_addr);  // free entry
	    }
	
    {- -------------------------------------------
  (1.1) コード生成:
        「もし BasicObjectLock の _lock フィールドが NULL であれば 
          (= これは自分が再帰的にロックを取得していたケース), 
         単にその BasicObjectLock を開放するだけで終了 (最後尾にある done までジャンプ)」
        ---------------------------------------- -}

	    // Test first if we are in the fast recursive case
	    Address lock_addr(lock_reg, BasicObjectLock::lock_offset_in_bytes() + BasicLock::displaced_header_offset_in_bytes());
	    ld_ptr(lock_addr, displaced_header_reg);
	    br_null(displaced_header_reg, true, Assembler::pn, done);
	    delayed()->st_ptr(G0, lockobj_addr);  // free entry
	
    {- -------------------------------------------
  (1.1) コード生成:
        「もし軽量ロック(stack-locked)であれば, 待避していた mark フィールドの中身を CAS で書き戻す. 
          成功したら終了 (done までジャンプ)
  
          (正確に言うと, 自分が書き換えたときのままだと想定していきなり CAS する. 
          成功してたら自分が書き換えたときのままだった(= stack-lockedだった)ということになる)」
  
         (なお, 以下の "if (!UseBiasedLocking)"  の箇所は, 
         UseBiasedLocking 時には上でやってしまっている処理を行っている模様. 
         if でくくらないと UseBiasedLocking 時には 
         load を2回行うことになって無駄, というだけのことだろう.)
        ---------------------------------------- -}

	    // See if it is still a light weight lock, if so we just unlock
	    // the object and we are done
	
	    if (!UseBiasedLocking) {
	      // load the object out of the BasicObjectLock
	      ld_ptr(lockobj_addr, obj_reg);
	    }
	
	    // we have the displaced header in displaced_header_reg
	    // we expect to see the stack address of the basicLock in case the
	    // lock is still a light weight lock (lock_reg)
	    assert(mark_addr.disp() == 0, "cas must take a zero displacement");
	    casx_under_lock(mark_addr.base(), lock_reg, displaced_header_reg,
	      (address)StubRoutines::Sparc::atomic_memory_operation_lock_addr());
	    cmp(lock_reg, displaced_header_reg);
	    brx(Assembler::equal, true, Assembler::pn, done);
	    delayed()->st_ptr(G0, lockobj_addr);  // free entry
	
    {- -------------------------------------------
  (1.1) コード生成:
        「CAS が失敗した場合には, 
        InterpreterRuntime::monitorexit() を呼び出して slow-path のコードにフォールバック」
        ---------------------------------------- -}

	    // The lock has been converted into a heavy lock and hence
	    // we need to get into the slow case
	
	    call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::monitorexit), lock_reg);
	
    {- -------------------------------------------
  (1.1) (ここが done ラベルの位置)
        ---------------------------------------- -}

	    bind(done);
	  }
	}
	
```


