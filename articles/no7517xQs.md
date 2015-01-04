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
void instanceKlass::set_initialization_state_and_notify_impl(instanceKlassHandle this_oop, ClassState state, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) notify_all() を呼び出す必要があるので, ここでロックを取っておく.
      ---------------------------------------- -}

	  ObjectLocker ol(this_oop, THREAD);

  {- -------------------------------------------
  (1) このクラスオブジェクトの初期化状態を 
      state 引数で指定されたものに変更する
      ---------------------------------------- -}

	  this_oop->set_init_state(state);

  {- -------------------------------------------
  (1) 初期化処理が終わったため, 
      初期化処理で排他待ちしている他スレッドを起こしておく.
      (See: instanceKlass::initialize_impl())
      ---------------------------------------- -}

	  ol.notify_all(CHECK);
	}
	
```


