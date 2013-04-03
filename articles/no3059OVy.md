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
void TemplateTable::_return(TosState state) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(state, state);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_desc->calls_vm(), "inconsistent calls_vm information");
	
  {- -------------------------------------------
  (1) コード生成: (ただし, return_register_finalizer バイトコード用のテンプレートの場合のみ.
                 その他の *return バイトコードの場合には, 以下のコードは生成しない)
      「クラスの access flags 中に JVM_ACC_HAS_FINALIZER ビットが立っているかどうかを調べる.
        もし立っていれば, InterpreterRuntime::register_finalizer() を呼び出して Finalizer の登録処理を行っておく.
        (ビットが立っていなければ, 呼び出しは省略する)」
  
      (なお, return_register_finalizer は rewrite 処理によって生成されるバイトコード.  (See: [here](no3059AfB.html) for details))
      ---------------------------------------- -}

	  if (_desc->bytecode() == Bytecodes::_return_register_finalizer) {
	    assert(state == vtos, "only valid state");
	    __ mov(G0, G3_scratch);
	    __ access_local_ptr(G3_scratch, Otos_i);
	    __ load_klass(Otos_i, O2);
	    __ set(JVM_ACC_HAS_FINALIZER, G3);
	    __ ld(O2, Klass::access_flags_offset_in_bytes() + sizeof(oopDesc), O2);
	    __ andcc(G3, O2, G0);
	    Label skip_register_finalizer;
	    __ br(Assembler::zero, false, Assembler::pn, skip_register_finalizer);
	    __ delayed()->nop();
	
	    // Call out to do finalizer registration
	    __ call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::register_finalizer), Otos_i);
	
	    __ bind(skip_register_finalizer);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::remove_activation() が生成するコードで
        スタックフレームを破棄するための準備を行う.
        synchronized method であればロックの解除も行う.」
      ---------------------------------------- -}

	  __ remove_activation(state, /* throw_monitor_exception */ true);
	
  {- -------------------------------------------
  (1) コード生成:
      「restore 命令でレジスタを復帰(SP も I5_savedSP に復帰)させつつ, リターンする」
      ---------------------------------------- -}

	  // The caller's SP was adjusted upon method entry to accomodate
	  // the callee's non-argument locals. Undo that adjustment.
	  __ ret();                             // return to caller
	  __ delayed()->restore(I5_savedSP, G0, SP);
	}
	
```


