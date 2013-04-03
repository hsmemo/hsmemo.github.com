---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/stubGenerator_x86_64.cpp

### 名前(function name)
```
  StubGenerator(CodeBuffer* code, bool all) : StubCodeGenerator(code) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下のどちらかを呼び出して初期化処理を行う.
      
      * all 引数が true の場合:  StubGenerator::generate_all()
      * all 引数が false の場合: StubGenerator::generate_initial()
      ---------------------------------------- -}

	    if (all) {
	      generate_all();
	    } else {
	      generate_initial();
	    }
	  }
	
```


