---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/methodOop.cpp

### 名前(function name)
```
void BreakpointInfo::clear(methodOop method) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コンストラクタ引数で指定された箇所を元のバイトコードに戻す.
      ---------------------------------------- -}

	  *method->bcp_from(_bci) = orig_bytecode();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(method->number_of_breakpoints() > 0, "must not go negative");

  {- -------------------------------------------
  (1) number_of_breakpoints を減らす
      (この値が 0 より大きいメソッドは JIT compile の対象にならない.
       (See: methodOopDesc::is_not_compilable()))
  
      (#TODO 他にも影響することはあるか？)
      ---------------------------------------- -}

	  method->decr_number_of_breakpoints();
	}
	
```


