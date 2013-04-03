---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/constantPoolOop.hpp

### 名前(function name)
```
  void method_handle_index_at_put(int which, int ref_kind, int ref_index) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) tags 配列(格納している各要素の種別を示す)に JVM_CONSTANT_MethodHandle を設定し, 
      さらに constantPoolOop オブジェクトの末尾(のフィールド宣言もされてない領域)に ref_* 引数の値を登録する.
      ---------------------------------------- -}

	    tag_at_put(which, JVM_CONSTANT_MethodHandle);
	    *int_at_addr(which) = ((jint) ref_index<<16) | ref_kind;
	  }
	
```


