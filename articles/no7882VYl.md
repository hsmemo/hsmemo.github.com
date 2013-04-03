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
void TemplateInterpreterGenerator::set_entry_points_for_all_bytes() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) dispatch table の全エントリに対して,  (<= DispatchTable::length 個分だけ存在)
      以下のようなコードをセットする.
      * bytecode として有効なエントリであれば (= Bytecodes::is_defined() が true であれば)
        TemplateInterpreterGenerator::set_entry_points() で対応するコードを生成し, その結果をセットする.
      * bytecode として有効なエントリでなければ (= Bytecodes::is_defined() が false であれば)
        TemplateInterpreterGenerator::set_unimplemented() で異常終了するコードを生成する.
        (See: TemplateInterpreterGenerator::set_unimplemented())
      ---------------------------------------- -}

	  for (int i = 0; i < DispatchTable::length; i++) {
	    Bytecodes::Code code = (Bytecodes::Code)i;
	    if (Bytecodes::is_defined(code)) {
	      set_entry_points(code);
	    } else {
	      set_unimplemented(i);
	    }
	  }
	}
	
```


