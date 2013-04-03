---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interp_masm_sparc.cpp

### 名前(function name)
```
void InterpreterMacroAssembler::call_VM_base(
  Register        oop_result,
  Register        java_thread,
  Register        last_java_sp,
  address         entry_point,
  int             number_of_arguments,
  bool            check_exception
) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (引数でカレントスレッドの JavaThread オブジェクトを指したレジスタ(java_thread)が指定されていなければ, 
       代わりに L7_thread_cache を使うことにする.)
  
      (といっても, 指定されていることがないような気もするが... #TODO)
      ---------------------------------------- -}

	  if (!java_thread->is_valid())
	    java_thread = L7_thread_cache;

  {- -------------------------------------------
  (1) コード生成:
      「MacroAssembler::call_VM_base() が生成するコードで呼び出しを行うだけ」
    
      (sparc では特に退避復帰の必要性はない?? 退避復帰コードがコメントアウトされている)
      ---------------------------------------- -}

	  // See class ThreadInVMfromInterpreter, which assumes that the interpreter
	  // takes responsibility for setting its own thread-state on call-out.
	  // However, ThreadInVMfromInterpreter resets the state to "in_Java".
	
	  //save_bcp();                                  // save bcp
	  MacroAssembler::call_VM_base(oop_result, java_thread, last_java_sp, entry_point, number_of_arguments, check_exception);
	  //restore_bcp();                               // restore bcp
	  //restore_locals();                            // restore locals pointer
	}
	
```


