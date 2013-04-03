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
void InterpreterMacroAssembler::dispatch_normal(TosState state) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Interpreter::normal_table() で TosState に応じた dispatch table を取得し, 
      それに基づいて以下のようなコードを生成:
      「InterpreterMacroAssembler::dispatch_base() が生成するコードにより, 
        適切な次のテンプレートのエントリポイントアドレスを計算して, そこにジャンプする」
      ---------------------------------------- -}

	  dispatch_base(state, Interpreter::normal_table(state));
	}
	
```


