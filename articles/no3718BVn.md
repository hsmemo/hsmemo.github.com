---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.cpp
### 説明(description)

```
// split the store check operation so that other instructions can be scheduled inbetween
```

### 名前(function name)
```
void MacroAssembler::store_check_part_1(Register obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 書き込んだアドレスに対応する card table の index を求める.
      (書き込んだアドレスを CardTableModRefBS::card_shift 分だけ右シフトする)
      ---------------------------------------- -}

	  BarrierSet* bs = Universe::heap()->barrier_set();
	  assert(bs->kind() == BarrierSet::CardTableModRef, "Wrong barrier set kind");
	  shrptr(obj, CardTableModRefBS::card_shift);
	}
	
```


