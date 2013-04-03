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
void TemplateTable::monitorexit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(atos, vtos);

  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_oop(Otos_i);

  {- -------------------------------------------
  (1) コード生成:
      「もしロック確保対象のオブジェクトが null であれば, NullPointerException」
      ---------------------------------------- -}

	  __ tst(Otos_i);
	  __ throw_if_not_x( Assembler::notZero, Interpreter::_throw_NullPointerException_entry, G3_scratch );
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(O0 == Otos_i, "just checking");
	
  {- -------------------------------------------
  (1) コード生成:
      「スタックフレーム中の BasicObjectLock 領域から対応する BasicObjectLock を探す.
        (見つからなければ InterpreterRuntime::throw_illegal_monitor_state_exception() を呼んで
         IllegalMonitorStateException を出す)」
      ---------------------------------------- -}

	  { Label entry, loop, found;
	    __ add( __ top_most_monitor(), O2 ); // last one to check
	    __ ba(false, entry );
	    // use Lscratch to hold monitor elem to check, start with most recent monitor,
	    // By using a local it survives the call to the C routine.
	    __ delayed()->mov( Lmonitors, Lscratch );
	
    {- -------------------------------------------
  (1.1) (ここからが, 対応する BasicObjectLock を探すループ)
        ---------------------------------------- -}

	    __ bind( loop );
	
	    __ verify_oop(O4);          // verify each monitor's oop
	    __ cmp(O4, O0); // check if current entry is for desired object
	    __ brx( Assembler::equal, true, Assembler::pt, found );
	    __ delayed()->mov(Lscratch, O1); // pass found entry as argument to monitorexit
	
	    __ inc( Lscratch, frame::interpreter_frame_monitor_size() * wordSize ); // advance to next
	
	    __ bind( entry );
	
	    __ cmp( Lscratch, O2 );
	    __ brx( Assembler::lessEqualUnsigned, true, Assembler::pt, loop );
	    __ delayed()->ld_ptr(Lscratch, BasicObjectLock::obj_offset_in_bytes(), O4);
	
    {- -------------------------------------------
  (1.1) (ここまでが探索のループ)
        ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) (見つからなければ InterpreterRuntime::throw_illegal_monitor_state_exception())
        ---------------------------------------- -}

	    call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::throw_illegal_monitor_state_exception));
	    __ should_not_reach_here();
	
	    __ bind(found);
	  }

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::unlock_object() が生成するコードで, 対象のオブジェクトにアンロック処理を行う」
      ---------------------------------------- -}

	  __ unlock_object(O1);
	}
	
```


