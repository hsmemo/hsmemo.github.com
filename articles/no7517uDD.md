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
void instanceKlass::call_class_initializer_impl(instanceKlassHandle this_oop, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  methodHandle h_method(THREAD, this_oop->class_initializer());

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!this_oop->is_initialized(), "we cannot initialize twice");

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceClassInitialization) {
	    tty->print("%d Initializing ", call_class_initializer_impl_counter++);
	    this_oop->name()->print_value();
	    tty->print_cr("%s (" INTPTR_FORMAT ")", h_method() == NULL ? "(no method)" : "", (address)this_oop());
	  }

  {- -------------------------------------------
  (1) もし static 初期化子があれば, 実行する.
      ---------------------------------------- -}

	  if (h_method() != NULL) {
	    JavaCallArguments args; // No arguments
	    JavaValue result(T_VOID);
	    JavaCalls::call(&result, h_method, &args, CHECK); // Static call (no args)
	  }
	}
	
```


