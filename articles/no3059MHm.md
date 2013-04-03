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
InterpreterGenerator::InterpreterGenerator(StubQueue* code)
  : TemplateInterpreterGenerator(code) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) スーパークラスである TemplateInterpreterGenerator のコンストラクタを呼び出した後, 
      TemplateInterpreterGenerator::generate_all() を呼び出して
      インタープリタの構築を行う.
      ---------------------------------------- -}

	   generate_all(); // down here so it can be "virtual"
	}
	
```


