---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEventController.cpp
### 説明(description)

```
// This change must always be occur when at a safepoint.
// Being at a safepoint causes the interpreter to use the
// safepoint dispatch table which we overload to find single
// step points.  Just to be sure that it has been set, we
// call notice_safepoints when turning on single stepping.
// When we leave our current safepoint, should_post_single_step
// will be checked by the interpreter, and the table kept
// or changed accordingly.
```

### 名前(function name)
```
void VM_ChangeSingleStep::doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiEventControllerPrivate::set_should_post_single_step() を呼んで, 
      JvmtiExport::_should_post_single_step を変更する.
      (なお, VM operation からでは変更できないので JvmtiEventControllerPrivate を経由しているらしい)
      ---------------------------------------- -}

	  JvmtiEventControllerPrivate::set_should_post_single_step(_on);

  {- -------------------------------------------
  (1) SingleStep イベントが有効化された場合には, 
      Interpreter::notice_safepoints() (をサブクラスがオーバーライドしたもの) を呼んでインタープリタに通知する.
      (VM operation なのに, その中でもう一度 Interpreter::notice_safepoints() を呼ぶ必要が有るのか?? #TODO)
      ---------------------------------------- -}

	  if (_on) {
	    Interpreter::notice_safepoints();
	  }
	}
	
```


