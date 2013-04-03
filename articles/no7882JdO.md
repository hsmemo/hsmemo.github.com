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
void InterpreterMacroAssembler::get_cache_index_at_bcp(Register cache, Register tmp,
                                                       int bcp_offset, size_t index_size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(bcp_offset > 0, "bcp is still pointing to start of bytecode");

  {- -------------------------------------------
  (1) コード生成:
      「バイトコード命令中に埋め込まれている index 情報を取得する」
      (なお, 引数で指定された大きさ(index_size) に応じて, 適切な大きさ分だけロードするコードを生成)
      ---------------------------------------- -}

	  if (index_size == sizeof(u2)) {
	    get_2_byte_integer_at_bcp(bcp_offset, cache, tmp, Unsigned);
	  } else if (index_size == sizeof(u4)) {
	    assert(EnableInvokeDynamic, "giant index used only for JSR 292");
	    get_4_byte_integer_at_bcp(bcp_offset, cache, tmp);
	    assert(constantPoolCacheOopDesc::decode_secondary_index(~123) == 123, "else change next line");
	    xor3(tmp, -1, tmp);  // convert to plain index
	  } else if (index_size == sizeof(u1)) {
	    assert(EnableInvokeDynamic, "tiny index used only for JSR 292");
	    ldub(Lbcp, bcp_offset, tmp);
	  } else {
	    ShouldNotReachHere();
	  }
	}
	
```


