---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp

### 名前(function name)
```
address TemplateInterpreterGenerator::generate_earlyret_entry_for(TosState state) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();
	
  {- -------------------------------------------
  (1) コード生成:
      「オペランドスタック("expression stack")を空にする.」
      ---------------------------------------- -}

	  __ empty_expression_stack();

  {- -------------------------------------------
  (1) コード生成:
      「ForceEarlyReturn*() で指定された返り値を TOS にロードする.」
      ---------------------------------------- -}

	  __ load_earlyret_value(state);
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドに対応する JvmtiThreadState オブジェクトを取得し,  
        その JvmtiThreadState::_earlyret_state フィールドをクリアする.」
      ---------------------------------------- -}

	  __ ld_ptr(G2_thread, JavaThread::jvmti_thread_state_offset(), G3_scratch);
	  Address cond_addr(G3_scratch, JvmtiThreadState::earlyret_state_offset());
	
	  // Clear the earlyret state
	  __ stw(G0 /* JvmtiThreadState::earlyret_inactive */, cond_addr);
	
  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::remove_activation() が生成するコードで
        スタックフレームを破棄する.
        synchronized method であればロックの解除も行う」
      ---------------------------------------- -}

	  __ remove_activation(state,
	                       /* throw_monitor_exception */ false,
	                       /* install_monitor_exception */ false);
	
  {- -------------------------------------------
  (1) コード生成:
      「restore 命令でレジスタを復帰(SP も I5_savedSP に復帰)させつつ, リターンする.
        (restore で ForceEarlyReturn*() 対象になったフレームを破棄).」
      ---------------------------------------- -}

	  // The caller's SP was adjusted upon method entry to accomodate
	  // the callee's non-argument locals. Undo that adjustment.
	  __ ret();                             // return to caller
	  __ delayed()->restore(I5_savedSP, G0, SP);
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	} // end of JVMTI ForceEarlyReturn support
	
```


