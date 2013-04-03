---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/templateTable.cpp

### 名前(function name)
```
void Template::initialize(int flags, TosState tos_in, TosState tos_out, generator gen, int arg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの初期化を行うだけ.
      (なお, ここで _gen フィールドに格納した関数に実際のコード生成処理が実装されている.
       See: Template::generate())
      ---------------------------------------- -}

	  _flags   = flags;
	  _tos_in  = tos_in;
	  _tos_out = tos_out;
	  _gen     = gen;
	  _arg     = arg;
	}
	
```


