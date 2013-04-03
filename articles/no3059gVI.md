---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.hpp
### 説明(description)
  // Stack overflow checking


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
  (1) 現在の SP から指定されたオフセットだけ離れたアドレスに rax レジスタの値を書き込むだけ.
      ---------------------------------------- -}

	    movl(Address(rsp, (-offset)), rax);
	  }
	
```


