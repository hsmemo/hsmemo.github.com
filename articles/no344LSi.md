---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vm_operations.cpp

### 名前(function name)
```
void VM_Operation::evaluate() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm;

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceVMOperation) {
	    tty->print("[");
	    NOT_PRODUCT(print();)
	  }

  {- -------------------------------------------
  (1) VM_Operation::doit() (を各サブクラスがオーバーライドしたもの) を呼び出す
      ---------------------------------------- -}

	  doit();

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceVMOperation) {
	    tty->print_cr("]");
	  }
	}
	
```


