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
void InterpreterMacroAssembler::get_cache_and_index_at_bcp(Register cache,
                                                           Register index,
                                                           int bcp_offset,
                                                           size_t index_size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(cache != index, "must use different registers");

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::get_cache_index_at_bcp() が生成したコードで, 
        現在箇所のバイトコードが参照している index を取得する (これは, バイトコード命令中に埋め込まれている情報を取ってくるだけ).
        その後, それをシフトし, index 引数で指定されたレジスタに格納する.
  
        また, ConstantPoolCache のアドレスをスタックフレーム上からロードし, 
        (これはスタックフレーム中の frame::interpreter_frame_cache_offset の位置にある)
        cache 引数で指定されたレジスタに格納する.」
      ---------------------------------------- -}

	  get_cache_index_at_bcp(index, bcp_offset, index_size);
	  movptr(cache, Address(rbp, frame::interpreter_frame_cache_offset * wordSize));
	  assert(sizeof(ConstantPoolCacheEntry) == 4 * wordSize, "adjust code below");
	  // convert from field index to ConstantPoolCacheEntry index
	  shll(index, 2);
	}
	
```


