---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) ThreadStateTransition::transition_from_native() を呼び出して, JavaThreadState を変更する.
      (Safepoint 処理が開始されていた場合は, この中でブロックする)
      ---------------------------------------- -}

	   void trans_from_native(JavaThreadState to)            { transition_from_native(_thread, to); }
	
```


