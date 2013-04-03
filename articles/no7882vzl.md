---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interp_masm_sparc.cpp
### 説明(description)

```
// Dispatch code executed in the prolog of a bytecode which does not do it's
// own dispatch. The dispatch address is computed and placed in IdispatchAddress
```

### 名前(function name)
```
void InterpreterMacroAssembler::dispatch_prolog(TosState state, int bcp_incr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_not_delayed();

  {- -------------------------------------------
  (1) コード生成:
      「次のバイトコード, および実行後の TosState に基づいて
        dispatch table から次のバイトコードに対応するテンプレートのエントリポイントアドレスを取得し, 
        IdispatchAddress レジスタに格納する」
      ---------------------------------------- -}

	#ifdef FAST_DISPATCH
	  // FAST_DISPATCH and ProfileInterpreter are mutually exclusive since
	  // they both use I2.
	  assert(!ProfileInterpreter, "FAST_DISPATCH and +ProfileInterpreter are mutually exclusive");
	  ldub(Lbcp, bcp_incr, Lbyte_code);                     // load next bytecode
	  add(Lbyte_code, Interpreter::distance_from_dispatch_table(state), Lbyte_code);
	                                                        // add offset to correct dispatch table
	  sll(Lbyte_code, LogBytesPerWord, Lbyte_code);         // multiply by wordSize
	  ld_ptr(IdispatchTables, Lbyte_code, IdispatchAddress);// get entry addr
	#else
	  ldub( Lbcp, bcp_incr, Lbyte_code);                    // load next bytecode
	  // dispatch table to use
	  AddressLiteral tbl(Interpreter::dispatch_table(state));
	  sll(Lbyte_code, LogBytesPerWord, Lbyte_code);         // multiply by wordSize
	  set(tbl, G3_scratch);                                 // compute addr of table
	  ld_ptr(G3_scratch, Lbyte_code, IdispatchAddress);     // get entry addr
	#endif
	}
	
```


