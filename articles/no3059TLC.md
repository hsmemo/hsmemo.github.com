---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/assembler_sparc.hpp
### 説明(description)
  // Note: this clobbers G3_scratch


### 名前(function name)
```
  void bang_stack_with_offset(int offset) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    // stack grows down, caller passes positive offset
	    assert(offset > 0, "must bang with negative offset");

  {- -------------------------------------------
  (1) 現在の SP から指定されたオフセット(+STACK_BIAS)のアドレスに 0 を書き込むだけ
      ---------------------------------------- -}

	    set((-offset)+STACK_BIAS, G3_scratch);
	    st(G0, SP, G3_scratch);
	  }
	
```


