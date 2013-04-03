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
IRT_ENTRY(void, InterpreterRuntime::throw_ClassCastException(
  JavaThread* thread, oopDesc* obj))
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(thread);
	  char* message = SharedRuntime::generate_class_cast_message(
	    thread, Klass::cast(obj->klass())->external_name());
	
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	  if (ProfileTraps) {
	    note_trap(thread, Deoptimization::Reason_class_check, CHECK);
	  }
	
  {- -------------------------------------------
  (1) THROW_MSG() で, 例外オブジェクトの生成と送出を行う.
      ---------------------------------------- -}

	  // create exception
	  THROW_MSG(vmSymbols::java_lang_ClassCastException(), message);
	IRT_END
	
```


