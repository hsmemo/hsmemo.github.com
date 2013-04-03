---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiThreadState.cpp

### 名前(function name)
```
void JvmtiThreadState::leave_interp_only_mode() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_thread->get_interp_only_mode() == 1, "leaving interp only when mode not one");

  {- -------------------------------------------
  (1) コンストラクタ引数で指定された JavaThread の _interp_only_mode フィールドの値をデクリメントするだけ.
      ---------------------------------------- -}

	  _thread->decrement_interp_only_mode();
	}
	
```


