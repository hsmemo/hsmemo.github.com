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
void ThreadService::add_thread(JavaThread* thread, bool daemon) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし処理対象が HotSpot が内部的に使用するスレッドだったり, 
      あるいは JVMTI の RunAgentThread() で使われるスレッドだった場合は, 
      特にすることはないので, ここでリターン.
      ---------------------------------------- -}

	  // Do not count VM internal or JVMTI agent threads
	  if (thread->is_hidden_from_external_view() ||
	      thread->is_jvmti_agent_thread()) {
	    return;
	  }
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) (See: UsePerfData, ThreadService)
      ("java.threads.daemon", "java.threads.live", "java.threads.livePeak", "java.threads.started")
      ---------------------------------------- -}

	  _total_threads_count->inc();
	  _live_threads_count->inc();
	
	  if (_live_threads_count->get_value() > _peak_threads_count->get_value()) {
	    _peak_threads_count->set_value(_live_threads_count->get_value());
	  }
	
	  if (daemon) {
	    _daemon_threads_count->inc();
	  }
	}
	
```


