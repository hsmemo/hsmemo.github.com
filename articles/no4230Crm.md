---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateTable_x86_64.cpp
### 説明(description)
(なお, スタックフレーム中の BasicObjectLock 領域のレイアウトは, 
 以下のアスキーアートのようになっているとのこと)

```
//-----------------------------------------------------------------------------
// Synchronization
//
// Note: monitorenter & exit are symmetric routines; which is reflected
//       in the assembly code structure as well
//
// Stack layout:
//
// [expressions  ] <--- rsp               = expression stack top
// ..
// [expressions  ]
// [monitor entry] <--- monitor block top = expression stack bot
// ..
// [monitor entry]
// [frame data   ] <--- monitor block bot
// ...
// [saved rbp    ] <--- rbp
```

### 名前(function name)
```
void TemplateTable::monitorenter() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(atos, vtos);
	
  {- -------------------------------------------
  (1) コード生成:
      「もしロック確保対象のオブジェクトが null であれば, NullPointerException」
      (See: [here](no30592Qc.html) for details)
      ---------------------------------------- -}

	  // check for NULL object
	  __ null_check(rax);
	
  {- -------------------------------------------
  (1) コード生成:
      「スタックフレーム中の BasicObjectLock 領域から空きスロットを探す」
      ---------------------------------------- -}

	  const Address monitor_block_top(
	        rbp, frame::interpreter_frame_monitor_block_top_offset * wordSize);
	  const Address monitor_block_bot(
	        rbp, frame::interpreter_frame_initial_sp_offset * wordSize);
	  const int entry_size = frame::interpreter_frame_monitor_size() * wordSize;
	
	  Label allocated;
	
	  // initialize entry pointer
	  __ xorl(c_rarg1, c_rarg1); // points to free slot or NULL
	
	  // find a free slot in the monitor block (result in c_rarg1)
	  {
	    Label entry, loop, exit;
	    __ movptr(c_rarg3, monitor_block_top); // points to current entry,
	                                     // starting with top-most entry
	    __ lea(c_rarg2, monitor_block_bot); // points to word before bottom
	                                     // of monitor block
	    __ jmpb(entry);
	
    {- -------------------------------------------
  (1.1) (ここからが, 空いている BasicObjectLock を探索するループ)
        ---------------------------------------- -}

	    __ bind(loop);
	    // check if current entry is used
	    __ cmpptr(Address(c_rarg3, BasicObjectLock::obj_offset_in_bytes()), (int32_t) NULL_WORD);
	    // if not used then remember entry in c_rarg1
	    __ cmov(Assembler::equal, c_rarg1, c_rarg3);
	    // check if current entry is for same object
	    __ cmpptr(rax, Address(c_rarg3, BasicObjectLock::obj_offset_in_bytes()));
	    // if same object then stop searching
	    __ jccb(Assembler::equal, exit);
	    // otherwise advance to next entry
	    __ addptr(c_rarg3, entry_size);
	    __ bind(entry);
	    // check if bottom reached
	    __ cmpptr(c_rarg3, c_rarg2);
	    // if not at bottom then check this entry
	    __ jcc(Assembler::notEqual, loop);

    {- -------------------------------------------
  (1.1) (ここまでが探索のループ)
        ---------------------------------------- -}

	    __ bind(exit);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「もし空きスロットがなければ, (既に入っているフレーム内のデータを 1つずつずらすことで) スペースを作る」
      ---------------------------------------- -}

	  __ testptr(c_rarg1, c_rarg1); // check if a slot has been found
	  __ jcc(Assembler::notZero, allocated); // if found, continue with that one
	
	  // allocate one if there's no free slot
	  {
	    Label entry, loop;
	    // 1. compute new pointers             // rsp: old expression stack top
	    __ movptr(c_rarg1, monitor_block_bot); // c_rarg1: old expression stack bottom
	    __ subptr(rsp, entry_size);            // move expression stack top
	    __ subptr(c_rarg1, entry_size);        // move expression stack bottom
	    __ mov(c_rarg3, rsp);                  // set start value for copy loop
	    __ movptr(monitor_block_bot, c_rarg1); // set new monitor block bottom
	    __ jmp(entry);
	    // 2. move expression stack contents
	    __ bind(loop);
	    __ movptr(c_rarg2, Address(c_rarg3, entry_size)); // load expression stack
	                                                      // word from old location
	    __ movptr(Address(c_rarg3, 0), c_rarg2);          // and store it at new location
	    __ addptr(c_rarg3, wordSize);                     // advance to next word
	    __ bind(entry);
	    __ cmpptr(c_rarg3, c_rarg1);            // check if bottom reached
	    __ jcc(Assembler::notEqual, loop);      // if not at bottom then
	                                            // copy next word
	  }
	
	  // call run-time routine
	  // c_rarg1: points to monitor entry
	  __ bind(allocated);
	
  {- -------------------------------------------
  (1) コード生成:
      「r13(bcp) を次のバイトコードに進めておく」
      ---------------------------------------- -}

	  // Increment bcp to point to the next bytecode, so exception
	  // handling for async. exceptions work correctly.
	  // The object has already been poped from the stack, so the
	  // expression stack looks correct.
	  __ increment(r13);
	
  {- -------------------------------------------
  (1) コード生成:
      「BasicObjectLock にロック対象のオブジェクトの oop を書き込む」
      ---------------------------------------- -}

	  // store object
	  __ movptr(Address(c_rarg1, BasicObjectLock::obj_offset_in_bytes()), rax);

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::lock_object() が生成するコードにより
        対象のオブジェクトにロック処理を行う」
      ---------------------------------------- -}

	  __ lock_object(c_rarg1);
	
  {- -------------------------------------------
  (1) コード生成:
      「InterpreterGenerator::generate_stack_overflow_check() が生成するコードにより
        stack overflow のチェックを行う」
      ---------------------------------------- -}

	  // check to make sure this monitor doesn't cause stack overflow after locking
	  __ save_bcp();  // in case of exception
	  __ generate_stack_overflow_check(0);
	
  {- -------------------------------------------
  (1) コード生成:
      「dispatch_next() が生成するコードで, 
        次のバイトコードに対応するテンプレートへとジャンプする」
      ---------------------------------------- -}

	  // The bcp has already been incremented. Just need to dispatch to
	  // next instruction.
	  __ dispatch_next(vtos);
	}
	
```


