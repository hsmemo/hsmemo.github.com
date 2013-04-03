---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp

### 名前(function name)
```
void ThreadStackTrace::dump_stack_at_safepoint(int maxDepth) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint(), "all threads are stopped");
	
  {- -------------------------------------------
  (1) コンストラクタ引数で指定されたスレッド(以下の _thread)に対して, 
      そのスタック中の全ての javaVFrame を辿り, 
      ThreadStackTrace::add_stack_frame() を呼んで
      スタックフレーム情報を _frames に追加する.
      ---------------------------------------- -}

	  if (_thread->has_last_Java_frame()) {
	    RegisterMap reg_map(_thread);
	    vframe* start_vf = _thread->last_java_vframe(&reg_map);
	    int count = 0;
	    for (vframe* f = start_vf; f; f = f->sender() ) {
	      if (f->is_java_frame()) {
	        javaVFrame* jvf = javaVFrame::cast(f);
	        add_stack_frame(jvf);
	        count++;
	      } else {
	        // Ignore non-Java frames
	      }
	      if (maxDepth > 0 && count == maxDepth) {
	        // Skip frames if more than maxDepth
	        break;
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) 「現在ロックされているオブジェクトモニターの一覧」も取得するようにと
      コンストラクタ引数で指定されていた場合 (_with_locked_monitors が true の場合), 
      InflatedMonitorsClosure を用いて
      ネイティブメソッド内がロックしたモニターについても情報を集めておく.
      ---------------------------------------- -}

	  if (_with_locked_monitors) {
	    // Iterate inflated monitors and find monitors locked by this thread
	    // not found in the stack
	    InflatedMonitorsClosure imc(_thread, this);
	    ObjectSynchronizer::monitors_iterate(&imc);
	  }
	}
	
```


