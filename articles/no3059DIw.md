---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp

### 名前(function name)
```
static instanceOop alloc_object(jclass clazz, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  KlassHandle k(THREAD, java_lang_Class::as_klassOop(JNIHandles::resolve_non_null(clazz)));

  {- -------------------------------------------
  (1) Klass::check_valid_for_instantiation を呼んで, 
      インスタンスを作れるクラスだと確認しておく.
      (Klass::check_valid_for_instantiation() もしくはサブクラスでオーバーライドしたメソッドが呼び出される)
      インスタンスが作れないクラスの場合には, ここで例外が出る.
      ---------------------------------------- -}

	  Klass::cast(k())->check_valid_for_instantiation(false, CHECK_NULL);

  {- -------------------------------------------
  (1) instanceKlass::initialize() で, リンク&初期化を(もしまだ行われていなければ)実行しておく?? #TODO
      ---------------------------------------- -}

	  instanceKlass::cast(k())->initialize(CHECK_NULL);

  {- -------------------------------------------
  (1) instanceKlass::allocate_instance() を呼んでメモリを確保する.
      ---------------------------------------- -}

	  instanceOop ih = instanceKlass::cast(k())->allocate_instance(THREAD);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return ih;
	}
	
```


