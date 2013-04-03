---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.hpp

### 名前(function name)
```
  AddressLiteral addr() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい AddressLiteral オブジェクトをリターンする.
  
      新しい AddressLiteral オブジェクトは this をコピーしたものだが, 
      _is_lval フィールドだけは true になっている.
      ---------------------------------------- -}

	    AddressLiteral ret = *this;
	    ret._is_lval = true;
	    return ret;
	  }
	
```


