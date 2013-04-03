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
BreakpointInfo::BreakpointInfo(methodOop m, int bci) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの初期化
    
      (_orig_bytecode フィールドに書き換え前の命令を記録している.
       なお置き換え前の命令は, まずは methodOopDesc::bcp_from() で取得を試みるが, 
       既に breakpoint が埋められていれば methodOopDesc::orig_bytecode_at() で取得する.
       こちらは全ての BreakpointInfo を iterate して元の命令を探すので少し重い.)
      (_next フィールドは最初は NULL (すぐにリストにつながれるが...))
      ---------------------------------------- -}

	  _bci = bci;
	  _name_index = m->name_index();
	  _signature_index = m->signature_index();
	  _orig_bytecode = (Bytecodes::Code) *m->bcp_from(_bci);
	  if (_orig_bytecode == Bytecodes::_breakpoint)
	    _orig_bytecode = m->orig_bytecode_at(_bci);
	  _next = NULL;
	}
	
```


