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
void TemplateInterpreterGenerator::set_unimplemented(int i) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) dispatch table 内の該当するエントリ(= Interpreter::_normal_table 及び Interpreter::_wentry_point[i]) を 
      TemplateInterpreterGenerator::_unimplemented_bytecode に格納されているコード (= 異常終了させるコード) で埋める.
  
      (See: TemplateInterpreterGenerator::_unimplemented_bytecode
            別名: InterpreterGenerator::_unimplemented_bytecode)
      ---------------------------------------- -}

	  address e = _unimplemented_bytecode;
	  EntryPoint entry(e, e, e, e, e, e, e, e, e);
	  Interpreter::_normal_table.set_entry(i, entry);
	  Interpreter::_wentry_point[i] = _unimplemented_bytecode;
	}
	
```


