---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/safepoint.cpp
### 説明(description)

```
// Returns true is thread could not be rolled forward at present position.
```

### 名前(function name)
```
void ThreadSafepointState::roll_forward(suspend_type type) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _type フィールドを変更する.
  
      (これにより, ThreadSafepointState::is_running() の値が変わる.
       See: ThreadSafepointState::is_running())
      ---------------------------------------- -}

	  _type = type;
	
  {- -------------------------------------------
  (1) SafepointSynchronize::signal_thread_at_safepoint() を呼んで, 
      _waiting_to_block を減少させておく.
      (See: SafepointSynchronize::signal_thread_at_safepoint())
  
  
      (ただし正確には, これは type 引数 (処理対象のスレッドの suspend_type) が _at_safepoint の場合の処理.
       それ以外の場合は以下のようになる.
       * type 引数が _call_back の場合:
         ThreadSafepointState::set_has_called_back() を呼び出すだけ.
         (これはデバッグ用の処理)
  
       * type 引数が _running の場合:
         この関数が呼ばれることはあり得ない. もし呼ばれたら ShouldNotReachHere())
      ---------------------------------------- -}

	  switch(_type) {
	    case _at_safepoint:
	      SafepointSynchronize::signal_thread_at_safepoint();
	      break;
	
	    case _call_back:
	      set_has_called_back(false);
	      break;
	
	    case _running:
	    default:
	      ShouldNotReachHere();
	  }
	}
	
```


