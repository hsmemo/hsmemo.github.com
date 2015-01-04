---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/instanceKlass.cpp

### 名前(function name)
```
void instanceKlass::call_class_initializer(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  instanceKlassHandle ik (THREAD, as_klassOop());

  {- -------------------------------------------
  (1) instanceKlass::call_class_initializer_impl() を呼び出すだけ.
      ---------------------------------------- -}

	  call_class_initializer_impl(ik, THREAD);
	}
	
```


