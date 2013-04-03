---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/templateInterpreter.cpp

### 名前(function name)
```
void TemplateInterpreterGenerator::set_entry_points(Bytecodes::Code code) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  CodeletMark cm(_masm, Bytecodes::name(code), code);
	  // initialize entry points

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_unimplemented_bytecode    != NULL, "should have been generated before");
	  assert(_illegal_bytecode_sequence != NULL, "should have been generated before");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  address bep = _illegal_bytecode_sequence;
	  address cep = _illegal_bytecode_sequence;
	  address sep = _illegal_bytecode_sequence;
	  address aep = _illegal_bytecode_sequence;
	  address iep = _illegal_bytecode_sequence;
	  address lep = _illegal_bytecode_sequence;
	  address fep = _illegal_bytecode_sequence;
	  address dep = _illegal_bytecode_sequence;
	  address vep = _unimplemented_bytecode;
	  address wep = _unimplemented_bytecode;

  {- -------------------------------------------
  (1) wide prefix 付きではないバイトコードであれば, 
      TemplateInterpreterGenerator::set_short_entry_points() によって, 
      対応するコードを生成し, 各 TOS 状態用のエントリポイントアドレスを取得する.
      ---------------------------------------- -}

	  // code for short & wide version of bytecode
	  if (Bytecodes::is_defined(code)) {
	    Template* t = TemplateTable::template_for(code);
	    assert(t->is_valid(), "just checking");
	    set_short_entry_points(t, bep, cep, sep, aep, iep, lep, fep, dep, vep);
	  }

  {- -------------------------------------------
  (1) wide prefix 付きのバイトコードであれば, 
      TemplateInterpreterGenerator::set_wide_entry_point() によって, 
      対応するコードを生成し, エントリポイントアドレスを取得する.
      ---------------------------------------- -}

	  if (Bytecodes::wide_is_defined(code)) {
	    Template* t = TemplateTable::template_for_wide(code);
	    assert(t->is_valid(), "just checking");
	    set_wide_entry_point(t, wep);
	  }

  {- -------------------------------------------
  (1) 取得したエントリポイントを, dispatch table 内の該当するエントリにセットする.
      (= Interpreter::_normal_table 及び Interpreter::_wentry_point[code] にセット).
      ---------------------------------------- -}

	  // set entry points
	  EntryPoint entry(bep, cep, sep, aep, iep, lep, fep, dep, vep);
	  Interpreter::_normal_table.set_entry(code, entry);
	  Interpreter::_wentry_point[code] = wep;
	}
	
```


