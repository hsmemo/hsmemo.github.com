---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/nativeInst_x86.hpp

### 名前(function name)
```
inline NativeJump* nativeJump_at(address address) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) address 引数の値をそのままリターンするだけ.
  
      (正確には NativeJump::instruction_offset を引いているが, これは 0 なので実質何もしていない.
       というか, オペコード(instruction)が先頭にあるのはほぼ自明なので, 
       引く意味はあまりない気もするが...(まぁ一応 prefix があるけど...) #TODO)
      ---------------------------------------- -}

	  NativeJump* jump = (NativeJump*)(address - NativeJump::instruction_offset);
	#ifdef ASSERT
	  jump->verify();
	#endif
	  return jump;
	}
	
```


