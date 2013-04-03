---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/iterator.cpp

### 名前(function name)
```
void ObjectToOopClosure::do_object(oop obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) oopDesc::oop_iterate() を呼んで, obj 引数内の全てのポインタフィールドに対して
      コンストラクタ引数で渡された OopClosure を実行するだけ.
      ---------------------------------------- -}

	  obj->oop_iterate(_cl);
	}
	
```


