---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/oopFactory.cpp

### 名前(function name)
```
typeArrayOop oopFactory::new_typeArray(BasicType type, int length, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  klassOop type_asKlassOop = Universe::typeArrayKlassObj(type);
	  typeArrayKlass* type_asArrayKlass = typeArrayKlass::cast(type_asKlassOop);

  {- -------------------------------------------
  (1) typeArrayKlass::allocate() でメモリを確保し, 結果をリターンする
      ---------------------------------------- -}

	  typeArrayOop result = type_asArrayKlass->allocate(length, THREAD);
	  return result;
	}
	
```


