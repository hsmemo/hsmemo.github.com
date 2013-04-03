---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/templateInterpreter.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) _active_table から次の命令を取得してリターンする.
      ---------------------------------------- -}

	  static address*   dispatch_table(TosState state)              { return _active_table.table_for(state); }
	
```


