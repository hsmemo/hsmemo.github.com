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
  void long_at_put(int which, jlong l) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) tags 配列(格納している各要素の種別を示す)に JVM_CONSTANT_Long を設定し, 
      さらに constantPoolOop オブジェクトの末尾(のフィールド宣言もされてない領域)に l 引数の値を登録する.
      ---------------------------------------- -}

	    tag_at_put(which, JVM_CONSTANT_Long);
	    // *long_at_addr(which) = l;
	    Bytes::put_native_u8((address)long_at_addr(which), *((u8*) &l));
	  }
	
```


