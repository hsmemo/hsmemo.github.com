---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp

### 名前(function name)
```
  ThreadBlockInVM(JavaThread *thread)
  : ThreadStateTransition(thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JavaFrameAnchor::make_walkable() を呼んで, スタックフレームを辿れるようにしておく.
      ---------------------------------------- -}

	    // Once we are blocked vm expects stack to be walkable
	    thread->frame_anchor()->make_walkable(thread);

  {- -------------------------------------------
  (1) ThreadStateTransition::trans_and_fence() を呼んで, JavaThreadState を _thread_blocked に変更する.
      ---------------------------------------- -}

	    trans_and_fence(_thread_in_vm, _thread_blocked);
	  }
	
```


