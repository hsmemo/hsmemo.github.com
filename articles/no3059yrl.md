---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/templateTable.cpp

### 名前(function name)
```
void TemplateTable::def(Bytecodes::Code code, int flags, TosState in, TosState out, void (*gen)(int arg), int arg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // should factor out these constants
	  const int ubcp = 1 << Template::uses_bcp_bit;
	  const int disp = 1 << Template::does_dispatch_bit;
	  const int clvm = 1 << Template::calls_vm_bit;
	  const int iswd = 1 << Template::wide_bit;
	  // determine which table to use
	  bool is_wide = (flags & iswd) != 0;
	  // make sure that wide instructions have a vtos entry point
	  // (since they are executed extremely rarely, it doesn't pay out to have an
	  // extra set of 5 dispatch tables for the wide instructions - for simplicity
	  // they all go with one table)
	  assert(in == vtos || !is_wide, "wide instructions have vtos entry point only");

  {- -------------------------------------------
  (1) 対応する Template オブジェクトを作成し, Template::initialize() でフィールドの初期化を行う.
      ---------------------------------------- -}

	  Template* t = is_wide ? template_for_wide(code) : template_for(code);
	  // setup entry
	  t->initialize(flags, in, out, gen, arg);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(t->bytecode() == code, "just checkin'");
	}
	
```


