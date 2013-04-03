---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interpreter_sparc.cpp
### 説明(description)

```
// Abstract method entry
// Attempt to execute abstract method. Throw exception
//
```

### 名前(function name)
```
address InterpreterGenerator::generate_abstract_entry(void) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();

  {- -------------------------------------------
  (1) コード生成:
      「AbstractMethodError を送出するだけ」
      ---------------------------------------- -}

	  // abstract method entry
	  // throw exception
	  __ call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::throw_AbstractMethodError));
	  // the call_VM checks for exception, so we should never return here.
	  __ should_not_reach_here();

  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	
	}
	
```


