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
void JvmtiThreadState::enter_interp_only_mode() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_thread->get_interp_only_mode() == 0, "entering interp only when mode not zero");

  {- -------------------------------------------
  (1) コンストラクタ引数で指定された JavaThread の _interp_only_mode フィールドの値をインクリメントするだけ.
      ---------------------------------------- -}

	  _thread->increment_interp_only_mode();
	}
	
```


