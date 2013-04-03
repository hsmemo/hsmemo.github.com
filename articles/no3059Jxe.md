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
void ThreadService::remove_thread(JavaThread* thread, bool daemon) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (プロファイル情報の記録) (JMM 用) (See: ThreadService::get_live_thread_count())
      ---------------------------------------- -}

	  Atomic::dec((jint*) &_exiting_threads_count);
	
  {- -------------------------------------------
  (1) もし処理対象が HotSpot が内部的に使用するスレッドだったり, 
      あるいは JVMTI の RunAgentThread() で使われるスレッドだった場合は, 
      以降のプロファイル情報には関係が無いので, ここでリターン.
      ---------------------------------------- -}

	  if (thread->is_hidden_from_external_view() ||
	      thread->is_jvmti_agent_thread()) {
	    return;
	  }
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) ("java.threads.live") (See: UsePerfData, ThreadService)
      ---------------------------------------- -}

	  _live_threads_count->set_value(_live_threads_count->get_value() - 1);
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) ("java.threads.daemon") (See: UsePerfData, ThreadService)
      及び
      (プロファイル情報の記録) (JMM 用) (See: ThreadService::get_daemon_thread_count())
      ---------------------------------------- -}

	  if (daemon) {
	    _daemon_threads_count->set_value(_daemon_threads_count->get_value() - 1);
	    Atomic::dec((jint*) &_exiting_daemon_threads_count);
	  }
	}
	
```


