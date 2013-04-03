---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interp_masm_x86_64.hpp

### 名前(function name)
```
  void save_bcp() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) r13 レジスタの値をフレーム上に待避するだけ (待避位置は frame::interpreter_frame_bcx_offset)
      ---------------------------------------- -}

	    movptr(Address(rbp, frame::interpreter_frame_bcx_offset * wordSize), r13);
	  }
	
```


