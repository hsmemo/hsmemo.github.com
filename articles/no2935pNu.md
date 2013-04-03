---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiThreadState.cpp
### 説明(description)

```
// Class:     JvmtiThreadState
// Function:  update_for_pop_top_frame
// Description:
//   This function removes any frame pop notification request for
//   the top frame and invalidates both the current stack depth and
//   all cached frameIDs.
//
// Called by: PopFrame
//
```

### 名前(function name)
```
void JvmtiThreadState::update_for_pop_top_frame() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以降で, スタックの先頭のフレームに設定されている NotifyFramePop() を全て破棄する処理を行う.
       ただし, この処理は is_interp_only_mode() が true でなければ行わない.
       というのは interp_only_mode になっていないなら
       そもそも JVMTI_EVENT_FRAME_POP イベント自体が有効化されていない(だからすることもない)はずなので.)
      ---------------------------------------- -}

	  if (is_interp_only_mode()) {

    {- -------------------------------------------
  (1.1) この JvmtiThreadState オブジェクト内の全ての JvmtiEnvThreadState を辿り, 
        現在のスタックフレームの深さに NotifyFramePop が仕掛けられているもの全てに対して, 
        JvmtiEnvThreadState::clear_frame_pop() を呼び出して
        その NotifyFramePop を破棄する.
        ---------------------------------------- -}

	    // remove any frame pop notification request for the top frame
	    // in any environment
	    int popframe_number = cur_stack_depth();
	    {
	      JvmtiEnvThreadStateIterator it(this);
	      for (JvmtiEnvThreadState* ets = it.first(); ets != NULL; ets = it.next(ets)) {
	        if (ets->is_frame_pop(popframe_number)) {
	          ets->clear_frame_pop(popframe_number);
	        }
	      }
	    }

    {- -------------------------------------------
  (1.1) JvmtiThreadState::invalidate_cur_stack_depth() を呼び出し, 
        JvmtiThreadState::_cur_stack_depth フィールドの値をリセットしておく.
    
        (これは FramePop イベントのための処理. (See: [here](no2935pZs.html) for details))
        ---------------------------------------- -}

	    // force stack depth to be recalculated
	    invalidate_cur_stack_depth();

  {- -------------------------------------------
  (1) (interp_only_mode になっていないなら JVMTI_EVENT_FRAME_POP イベント自体が有効化されていないはず)
      ---------------------------------------- -}

	  } else {
	    assert(!is_enabled(JVMTI_EVENT_FRAME_POP), "Must have no framepops set");
	  }
	}
	
```


