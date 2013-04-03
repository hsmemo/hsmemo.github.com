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
  void name_and_type_at_put(int which, int name_index, int signature_index) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) tags 配列(格納している各要素の種別を示す)に JVM_CONSTANT_NameAndType を設定し, 
      さらに constantPoolOop オブジェクトの末尾(のフィールド宣言もされてない領域)に *_index 引数の値を登録する.
      ---------------------------------------- -}

	    tag_at_put(which, JVM_CONSTANT_NameAndType);
	    *int_at_addr(which) = ((jint) signature_index<<16) | name_index;  // Not so nice
	  }
	
```


