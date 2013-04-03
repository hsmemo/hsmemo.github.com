---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_64.cpp

### 名前(function name)
```
void TemplateInterpreterGenerator::trace_bytecode(Template* t) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Call a little run-time stub to avoid blow-up for each bytecode.
	  // The run-time runtime saves the right registers, depending on
	  // the tosca in-state for the given template.
	
	  assert(Interpreter::trace_code(t->tos_in()) != NULL,
	         "entry must have been generated");

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  __ mov(r12, rsp); // remember sp (can only use r12 if not using call_VM)
	  __ andptr(rsp, -16); // align stack as required by ABI

  {- -------------------------------------------
  (1) Interpreter::trace_code() が指しているコード (= 
      TemplateInterpreterGenerator::generate_trace_code() で
      生成されたコード列のエントリポイント) を呼び出す.
      ---------------------------------------- -}

	  __ call(RuntimeAddress(Interpreter::trace_code(t->tos_in())));
	
```


