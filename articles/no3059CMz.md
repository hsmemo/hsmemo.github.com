---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateTable_sparc.cpp

### 名前(function name)
```
void TemplateTable::athrow() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(atos, vtos);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // This works because exception is cached in Otos_i which is same as O0,
	  // which is same as what throw_exception_entry_expects
	  assert(Otos_i == Oexception, "see explanation above");
	
  {- -------------------------------------------
  (1) コード生成: (verify)用
      ---------------------------------------- -}

	  __ verify_oop(Otos_i);

  {- -------------------------------------------
  (1) コード生成:
      「もし送出対象の例外オブジェクトが null であれば, NullPointerException」
      (See: [here](no30592Qc.html) for details)
      ---------------------------------------- -}

	  __ null_check(Otos_i);

  {- -------------------------------------------
  (1) コード生成:
      「Interpreter::throw_exception_entry() が指すコードへジャンプし, 例外送出処理を行う.」
      ---------------------------------------- -}

	  __ throw_if_not_x(Assembler::never, Interpreter::throw_exception_entry(), G3_scratch);
	}
	
```


