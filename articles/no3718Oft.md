---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.cpp

### 名前(function name)
```
void MacroAssembler::store_check_part_2(Register obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  BarrierSet* bs = Universe::heap()->barrier_set();
	  assert(bs->kind() == BarrierSet::CardTableModRef, "Wrong barrier set kind");
	  CardTableModRefBS* ct = (CardTableModRefBS*)bs;
	  assert(sizeof(*ct->byte_map_base) == sizeof(jbyte), "adjust this code");
	
  {- -------------------------------------------
  (1) movb で書き込んだアドレスに対応する card table のエントリに 0 を書き込む.
      (0 が dirty を示す (See: dirty_card))
    
      なお, 対応するエントリは以下のようにして算出 
      (ただし card_shift 分だけ右シフトする処理は MacroAssembler::store_check_part_1() で行っている)
        _byte_map - (uintptr_t(low_bound) >> card_shift)
      ---------------------------------------- -}

	  // The calculation for byte_map_base is as follows:
	  // byte_map_base = _byte_map - (uintptr_t(low_bound) >> card_shift);
	  // So this essentially converts an address to a displacement and
	  // it will never need to be relocated. On 64bit however the value may be too
	  // large for a 32bit displacement
	
	  intptr_t disp = (intptr_t) ct->byte_map_base;
	  if (is_simm32(disp)) {
	    Address cardtable(noreg, obj, Address::times_1, disp);
	    movb(cardtable, 0);
	  } else {
	    // By doing it as an ExternalAddress disp could be converted to a rip-relative
	    // displacement and done in a single instruction given favorable mapping and
	    // a smarter version of as_Address. Worst case it is two instructions which
	    // is no worse off then loading disp into a register and doing as a simple
	    // Address() as above.
	    // We can't do as ExternalAddress as the only style since if disp == 0 we'll
	    // assert since NULL isn't acceptable in a reloci (see 6644928). In any case
	    // in some cases we'll get a single instruction version.
	
	    ExternalAddress cardtable((address)disp);
	    Address index(noreg, obj, Address::times_1);
	    movb(as_Address(ArrayAddress(cardtable, index)), 0);
	  }
	}
	
```


