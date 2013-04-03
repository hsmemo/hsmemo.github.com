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
void Template::generate(InterpreterMacroAssembler* masm) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成時に使用するフィールドを初期化しておく
      ---------------------------------------- -}

	  // parameter passing
	  TemplateTable::_desc = this;
	  TemplateTable::_masm = masm;

  {- -------------------------------------------
  (1) Template::_gen フィールドに格納されているコード生成用の関数を呼び出す.
      (なお, _gen フィールドの値は TemplateTable::initialize() 内で初期化されており, 
       TemplateTable クラスの各メソッドが設定されている (TemplateTable::iconst(), TemplateTable::getstatic(), 等)
  
       詳細は, TemplateTable::initialize() 内での Template オブジェクト生成箇所の generator 欄を確認のこと.
       See: TemplateTable::initialize(), Template::initialize())
      ---------------------------------------- -}

	  // code generation
	  _gen(_arg);

  {- -------------------------------------------
  (1) 生成したコードから RuntimeStub を構築.
      ---------------------------------------- -}

	  masm->flush();
	}
	
```


