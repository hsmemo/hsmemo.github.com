---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/interpreter.cpp

### 名前(function name)
```
void interpreter_init() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Interpreter::initialize() を呼び出して, Interpreter を構築する.
      ---------------------------------------- -}

	  Interpreter::initialize();

  {- -------------------------------------------
  (1) (トレース出力用の処理) (See: BytecodeTracer)
      ---------------------------------------- -}

	#ifndef PRODUCT
	  if (TraceBytecodes) BytecodeTracer::set_closure(BytecodeTracer::std_closure());
	#endif // PRODUCT

  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (生成した stub を Forte::register_stub() で Forte に登録する)
      (See: Forte)
      ---------------------------------------- -}

	  // need to hit every safepoint in order to call zapping routine
	  // register the interpreter
	  Forte::register_stub(
	    "Interpreter",
	    AbstractInterpreter::code()->code_start(),
	    AbstractInterpreter::code()->code_end()
	  );
	
  {- -------------------------------------------
  (1) (JVMTI のフック点)
      ---------------------------------------- -}

	  // notify JVMTI profiler
	  if (JvmtiExport::should_post_dynamic_code_generated()) {
	    JvmtiExport::post_dynamic_code_generated("Interpreter",
	                                             AbstractInterpreter::code()->code_start(),
	                                             AbstractInterpreter::code()->code_end());
	  }
	}
	
```


