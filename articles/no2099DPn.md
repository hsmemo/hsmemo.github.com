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
  void int_at_put(int which, jint i) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) tags 配列(格納している各要素の種別を示す)に JVM_CONSTANT_Integer を設定し, 
      さらに constantPoolOop オブジェクトの末尾(のフィールド宣言もされてない領域)に i 引数の値を登録する.
      ---------------------------------------- -}

	    tag_at_put(which, JVM_CONSTANT_Integer);
	    *int_at_addr(which) = i;
	  }
	
```


