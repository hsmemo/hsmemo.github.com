---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiExport.cpp
### 説明(description)

```
// Todo: inline this for optimization
```

### 名前(function name)
```
void JvmtiExport::post_single_step(JavaThread *thread, methodOop method, address location) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  HandleMark hm(thread);
	  methodHandle mh(thread, method);
	
  {- -------------------------------------------
  (1) 処理対象のスレッドの JvmtiThreadState を取得する.
      JvmtiThreadState がまだ作られていなければ
      (明らかにこのタイミングでのイベント通知は不要なので) ここでリターン.
      ---------------------------------------- -}

	  JvmtiThreadState *state = thread->jvmti_thread_state();
	  if (state == NULL) {
	    return;
	  }

  {- -------------------------------------------
  (1) 処理対象の JvmtiThreadState オブジェクト内の全ての JvmtiEnvThreadState を辿り, 
      JVMTI_EVENT_SINGLE_STEP 通知が有効になっているもの全てに対してコールバックの呼び出しを行う.
  
      (なお正確に言うと, 二重にイベントを送信しないように 
       JvmtiEnvThreadState::compare_and_set_current_location() でチェックしている.
       二重送信の場合は single_stepping_posted() が true を返すようになる.
       二重送信ではなく, かつ JVMTI_EVENT_SINGLE_STEP が有効になっており, 
       さらにコールバックが設定されている JvmtiEnvThreadState に対してだけ, コールバック呼び出しが行われる.)
    
      (また, 一度コールバックを呼び出した JvmtiEnvThreadState についても, 
       JvmtiEnvThreadState::set_single_stepping_posted() を呼んで 
       single_stepping_posted() が true になるようにしている.)
  
      (ついでに, コールバックの呼び出しを行う場合には(トレース出力)も出している)
      ---------------------------------------- -}

	  JvmtiEnvThreadStateIterator it(state);
	  for (JvmtiEnvThreadState* ets = it.first(); ets != NULL; ets = it.next(ets)) {
	    ets->compare_and_set_current_location(mh(), location, JVMTI_EVENT_SINGLE_STEP);
	    if (!ets->single_stepping_posted() && ets->is_enabled(JVMTI_EVENT_SINGLE_STEP)) {
	      EVT_TRACE(JVMTI_EVENT_SINGLE_STEP, ("JVMTI [%s] Evt Single Step sent %s.%s @ %d",
	                    JvmtiTrace::safe_get_thread_name(thread),
	                    (mh() == NULL) ? "NULL" : mh()->klass_name()->as_C_string(),
	                    (mh() == NULL) ? "NULL" : mh()->name()->as_C_string(),
	                    location - mh()->code_base() ));
	
	      JvmtiEnv *env = ets->get_env();
	      JvmtiLocationEventMark jem(thread, mh, location);
	      JvmtiJavaThreadEventTransition jet(thread);
	      jvmtiEventSingleStep callback = env->callbacks()->SingleStep;
	      if (callback != NULL) {
	        (*callback)(env->jvmti_external(), jem.jni_env(), jem.jni_thread(),
	                    jem.jni_methodID(), jem.location());
	      }
	
	      ets->set_single_stepping_posted();
	    }
	  }
	}
	
```


