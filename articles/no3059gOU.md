---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/assembler_sparc.cpp

### 名前(function name)
```
void MacroAssembler::calc_mem_param_words(Register Rparam_words, Register Rresult) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「引数で指定された Rparam_words レジスタから 
        Argument::n_register_parameters 分だけ引いた値を計算し, 
        結果を (引数で指定された) Rresult レジスタに入れる.
  
        ただし, 結果が負数になっていたら, 代わりに 0 を結果とする」
    
       (レジスタで引き渡せる分はスタックを消費しないので, その分だけ減らしておくという処理)
      ---------------------------------------- -}

	  subcc( Rparam_words, Argument::n_register_parameters, Rresult); // how many mem words?
	  Label no_extras;
	  br( negative, true, pt, no_extras ); // if neg, clear reg
	  delayed()->set(0, Rresult);          // annuled, so only if taken
	  bind( no_extras );
	}
	
```


