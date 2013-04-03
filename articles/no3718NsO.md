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
void MacroAssembler::card_table_write(jbyte* byte_map_base,
                                      Register tmp, Register obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成: 
      「書き込み先のアドレスに対応する card table 中のエントリを dirty 化する(= 0 を書き込む).」
  
      (なお, 0 が dirty を示す (See: dirty_card))
  
      (より具体的には, 
       まず, 書き込み先のアドレス(obj)の値を CardTableModRefBS::card_shift 分だけ右シフトさせて, 
       対応する card table 内の index を計算する.
       次に, それに card table のアドレス(byte_map_base 引数の値)を足し込んで
       該当エントリのアドレスを計算し, 
       最後に, そこに stb で 0 を書き込む.)
      ---------------------------------------- -}

	#ifdef _LP64
	  srlx(obj, CardTableModRefBS::card_shift, obj);
	#else
	  srl(obj, CardTableModRefBS::card_shift, obj);
	#endif
	  assert(tmp != obj, "need separate temp reg");
	  set((address) byte_map_base, tmp);
	  stb(G0, tmp, obj);
	}
	
```


