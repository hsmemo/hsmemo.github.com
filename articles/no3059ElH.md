---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interpreterRT_x86_64.cpp

### 名前(function name)
```
void InterpreterRuntime::SignatureHandlerGenerator::generate(uint64_t fingerprint) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // generate code to handle arguments
	  iterate(fingerprint);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // return result handler
	  __ lea(rax, ExternalAddress(Interpreter::result_handler(method()->result_type())));
	  __ ret(0);
	
	  __ flush();
	}
	
```


