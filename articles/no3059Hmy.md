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
// remove activation
//
// Unlock the receiver if this is a synchronized method.
// Unlock any Java monitors from syncronized blocks.
// Remove the activation from the stack.
//
// If there are locked Java monitors
//    If throw_monitor_exception
//       throws IllegalMonitorStateException
//    Else if install_monitor_exception
//       installs IllegalMonitorStateException
//    Else
//       no error processing
```

### 名前(function name)
```
void InterpreterMacroAssembler::remove_activation(TosState state,
                                                  bool throw_monitor_exception,
                                                  bool install_monitor_exception) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::unlock_if_synchronized_method() が生成するコードで, 
       (もし synchronized メソッドの場合には) ロックを解放する.
       モニターのロック状態がおかしければ, IllegalMonitorStateException も送出される.
       (See: [here](no3059F5A.html) for details)
      ---------------------------------------- -}

	  unlock_if_synchronized_method(state, throw_monitor_exception, install_monitor_exception);
	
  {- -------------------------------------------
  (1) コード生成: (JVMTI のフック点)
      ---------------------------------------- -}

	  // save result (push state before jvmti call and pop it afterwards) and notify jvmti
	  notify_method_exit(false, state, NotifyJVMTI);
	
  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  interp_verify_oop(Otos_i, state, __FILE__, __LINE__);
	  verify_oop(Lmethod);
	  verify_thread();
	
  {- -------------------------------------------
  (1) コード生成:
      「返値を適切なレジスタに移動させておく」
    
      (この後, restore 命令によってレジスタウインドウが回ってしまうので, 
       restore 後でもきちんと見えるように移動させる.
       具体的には, O0 レジスタ (及び O1 レジスタ) に入っているものを 
       I0 レジスタ (及び I1 レジスタ) に移すだけ.
  
       引数で指定された tos (以下の state) に応じて, 適切な命令を生成している)
      ---------------------------------------- -}

	  // return tos
	  assert(Otos_l1 == Otos_i, "adjust code below");
	  switch (state) {
	#ifdef _LP64
	  case ltos: mov(Otos_l, Otos_l->after_save()); break; // O0 -> I0
	#else
	  case ltos: mov(Otos_l2, Otos_l2->after_save()); // fall through  // O1 -> I1
	#endif
	  case btos:                                      // fall through
	  case ctos:
	  case stos:                                      // fall through
	  case atos:                                      // fall through
	  case itos: mov(Otos_l1, Otos_l1->after_save());    break;        // O0 -> I0
	  case ftos:                                      // fall through
	  case dtos:                                      // fall through
	  case vtos: /* nothing to do */                     break;
	  default  : ShouldNotReachHere();
	  }
	
  {- -------------------------------------------
  (1) コード生成: (ただし, 32bit 環境でかつ C2 利用時でない場合には, 不要なので生成しない)
      「32bit 環境では long 値だけは扱いが特殊なので調整しておく」
      (正確に言うと, 32bit 環境では C2 とインタープリタで使うレジスタがずれている.
       C2 は G1 を使うがインタープリタは O0 と O1 を使う)
      ---------------------------------------- -}

	#if defined(COMPILER2) && !defined(_LP64)
	  if (state == ltos) {
	    // C2 expects long results in G1 we can't tell if we're returning to interpreted
	    // or compiled so just be safe use G1 and O0/O1
	
	    // Shift bits into high (msb) of G1
	    sllx(Otos_l1->after_save(), 32, G1);
	    // Zero extend low bits
	    srl (Otos_l2->after_save(), 0, Otos_l2->after_save());
	    or3 (Otos_l2->after_save(), G1, G1);
	  }
	#endif /* COMPILER2 */
	
	}
	
```


