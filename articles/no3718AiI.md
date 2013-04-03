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
void MacroAssembler::card_write_barrier_post(Register store_addr, Register new_val, Register tmp) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 書き込まれる値が G0 レジスタ (= 0) であれば, 
      記録用のコードを生成する必要は無いので, ここでリターン.
      ---------------------------------------- -}

	  // If we're writing constant NULL, we can skip the write barrier.
	  if (new_val == G0) return;

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  CardTableModRefBS* bs = (CardTableModRefBS*) Universe::heap()->barrier_set();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(bs->kind() == BarrierSet::CardTableModRef ||
	         bs->kind() == BarrierSet::CardTableExtension, "wrong barrier");

  {- -------------------------------------------
  (1) コード生成:
      「MacroAssembler::card_table_write() が生成するコードにより, 
        card table 内の該当エントリを dirty 化する.」
      ---------------------------------------- -}

	  card_table_write(bs->byte_map_base, tmp, store_addr);
	}
	
```


