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
instanceOop instanceKlass::register_finalizer(instanceOop i, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceFinalizerRegistration) {
	    tty->print("Registered ");
	    i->print_value_on(tty);
	    tty->print_cr(" (" INTPTR_FORMAT ") as finalizable", (address)i);
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  instanceHandle h_i(THREAD, i);
	  // Pass the handle as argument, JavaCalls::call expects oop as jobjects
	  JavaValue result(T_VOID);
	  JavaCallArguments args(h_i);
	  methodHandle mh (THREAD, Universe::finalizer_register_method());

  {- -------------------------------------------
  (1) JavaCalls::call() で java.lang.ref.Finalizer.register() メソッドを呼び出して
      新しい Finalizer オブジェクトを生成する.
      (See: java.lang.ref.Finalizer.register())
      ---------------------------------------- -}

	  JavaCalls::call(&result, mh, &args, CHECK_NULL);
	  return h_i();
	}
	
```


