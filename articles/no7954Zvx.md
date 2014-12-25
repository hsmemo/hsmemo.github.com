---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/objArrayKlass.cpp

### 名前(function name)
```
void objArrayKlass::initialize(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 配列要素のクラス(bottom_klass)の Klass::initialize() を呼び出す.
      (コメントによると, instanceKlass::initialize() か typeArrayKlass::initialize() のどちらか)
      ---------------------------------------- -}

	  Klass::cast(bottom_klass())->initialize(THREAD);  // dispatches to either instanceKlass or typeArrayKlass
	}
	
```


