---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/javaCalls.cpp

### 名前(function name)
```
void JavaCalls::call_virtual(JavaValue* result, KlassHandle spec_klass, Symbol* name, Symbol* signature, JavaCallArguments* args, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  CallInfo callinfo;
	  Handle receiver = args->receiver();
	  KlassHandle recvrKlass(THREAD, receiver.is_null() ? (klassOop)NULL : receiver->klass());

  {- -------------------------------------------
  (1) LinkResolver::resolve_virtual_call() を呼んで, 
      呼び出し先に対応する methodOop 情報を取得する 
      (以下の callinfo にセットされて返される).
  
      (なお, まだ resolve が終わっていなければ resolve も行う)
      ---------------------------------------- -}

	  LinkResolver::resolve_virtual_call(
	          callinfo, receiver, recvrKlass, spec_klass, name, signature,
	          KlassHandle(), false, true, CHECK);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  methodHandle method = callinfo.selected_method();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(method.not_null(), "should have thrown exception");
	
  {- -------------------------------------------
  (1) JavaCalls::call() で, 実際の呼び出し処理を行う.
      ---------------------------------------- -}

	  // Invoke the method
	  JavaCalls::call(result, method, args, CHECK);
	}
	
```


