---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/constantPoolOop.hpp
### 説明(description)
この関数が行うのは一時的な登録.

```
  // For temporary use while constructing constant pool
```

### 名前(function name)
```
  void klass_index_at_put(int which, int name_index) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) tags 配列(格納している各要素の種別を示す)に JVM_CONSTANT_ClassIndex を設定し, 
      さらに constantPoolOop オブジェクトの末尾(のフィールド宣言もされてない領域)に name_index 引数の値を登録する.
      ---------------------------------------- -}

	    tag_at_put(which, JVM_CONSTANT_ClassIndex);
	    *int_at_addr(which) = name_index;
	  }
	
```


