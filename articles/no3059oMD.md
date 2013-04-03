---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/interpreterRuntime.cpp

### 名前(function name)
```
IRT_ENTRY(void, InterpreterRuntime::create_exception(JavaThread* thread, char* name, char* message))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      (s は, 引数で指定された名前の例外クラスを SymbolTable 内から lookup したもの)
      ---------------------------------------- -}

	  // lookup exception klass
	  TempNewSymbol s = SymbolTable::new_symbol(name, CHECK);

  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	  if (ProfileTraps) {
	    if (s == vmSymbols::java_lang_ArithmeticException()) {
	      note_trap(thread, Deoptimization::Reason_div0_check, CHECK);
	    } else if (s == vmSymbols::java_lang_NullPointerException()) {
	      note_trap(thread, Deoptimization::Reason_null_check, CHECK);
	    }
	  }

  {- -------------------------------------------
  (1) Exceptions::new_exception() で例外オブジェクトを生成し, 
      引数で指定されたスレッドの vm_result フィールドに格納する.
      ---------------------------------------- -}

	  // create exception
	  Handle exception = Exceptions::new_exception(thread, s, message);
	  thread->set_vm_result(exception());
	IRT_END
	
```


