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
void TemplateInterpreterGenerator::set_safepoints_for_all_bytes() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Safepoint 停止用の dispatch table (TemplateInterpreter::_safept_table) の全エントリに対して
      (エントリは DispatchTable::length 個分だけ存在), 
      もしそれが bytecode として有効なエントリであれば (= Bytecodes::is_defined() が true であれば)
      Safeponit 停止させるコード (= TemplateInterpreter::_safept_entry に格納されているコード) をセットする. 
  
      (See: TemplateInterpreter::_safept_entry
            なお, TemplateInterpreter::_safept_entry は Interpreter::_safept_entry と書かれることもある.)
      ---------------------------------------- -}

	  for (int i = 0; i < DispatchTable::length; i++) {
	    Bytecodes::Code code = (Bytecodes::Code)i;
	    if (Bytecodes::is_defined(code)) Interpreter::_safept_table.set_entry(code, Interpreter::_safept_entry);
	  }
	}
	
```


