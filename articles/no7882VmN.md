---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interp_masm_x86_64.cpp

### 名前(function name)
```
void InterpreterMacroAssembler::dispatch_next(TosState state, int step) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「r13 から次のバイトコードを取得し, r13 をインクリメントする」
      ---------------------------------------- -}

	  // load next bytecode (load before advancing r13 to prevent AGI)
	  load_unsigned_byte(rbx, Address(r13, step));
	  // advance r13
	  increment(r13, step);

  {- -------------------------------------------
  (1) Interpreter::dispatch_table() で TosState に応じた dispatch table を取得し, 
      それに基づいて以下のようなコードを生成:
      「InterpreterMacroAssembler::dispatch_base() が生成するコードにより, 
        (rbx にロードした次のバイトコード, および取得した dispatch table に基づいて) 
        適切な次のテンプレートのエントリポイントアドレスを計算して, そこにジャンプする」
      ---------------------------------------- -}

	  dispatch_base(state, Interpreter::dispatch_table(state));
	}
	
```


