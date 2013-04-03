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

	  assert(_desc->calls_vm(),
	         "inconsistent calls_vm information"); // call in remove_activation
	
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
	    __ movptr(c_rarg1, aaddress(0));
	    __ load_klass(rdi, c_rarg1);
	    __ movl(rdi, Address(rdi, Klass::access_flags_offset_in_bytes() + sizeof(oopDesc)));
	    __ testl(rdi, JVM_ACC_HAS_FINALIZER);
	    Label skip_register_finalizer;
	    __ jcc(Assembler::zero, skip_register_finalizer);
	
	    __ call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::register_finalizer), c_rarg1);
	
	    __ bind(skip_register_finalizer);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::remove_activation() が生成するコードで
        スタックフレームを破棄する.
        synchronized method であればロックの解除も行う.
  
        (なお, この生成コードによってリターンアドレスが r13 にロードされる)」
      ---------------------------------------- -}

	  __ remove_activation(state, r13);

  {- -------------------------------------------
  (1) コード生成:
      「r13 に取得したリターンアドレスにジャンプする」
      ---------------------------------------- -}

	  __ jmp(r13);
	}
	
```


