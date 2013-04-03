---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvBase.cpp

### 名前(function name)
```
void
JvmtiEnvBase::env_dispose() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Threads::number_of_threads() == 0 || JvmtiThreadState_lock->is_locked(), "sanity check");
	
  {- -------------------------------------------
  (1) (この段階では全てのイベント通知が disabled になっている. そして enabled に戻ることはない.
       その理由は, この状態から enabled にするにはコールバックを設定する必要があるが, 
       コールバックを設定する関数内では JvmtiThreadState_lock を確保した状態で 
       JvmtiEnv が valid かどうかを確認した後に, 問題なければセットするため 
       (See: JvmtiEnvBase::set_event_callbacks()).
  
       (この関数自体は JvmtiThreadState_lock を確保した状態で呼ばれるため, コールバックの設定処理と競合することはない.
       また, 以降の処理で _magic を変更して JvmtiEnv を invalid な状態にしてしまうので, 
       JvmtiThreadState_lock を開放した後にコールバックがセットされることもない))
      ---------------------------------------- -}

	  // We have been entered with all events disabled on this environment.
	  // A race to re-enable events (by setting callbacks) is prevented by
	  // checking for a valid environment when setting callbacks (while
	  // holding the JvmtiThreadState_lock).
	
  {- -------------------------------------------
  (1) _magic フィールドを変更する.
      (これにより JvmtiEnvBase::is_valid() が false を返すようになり, 
       JvmtiEnvBase::set_event_callbacks() との race が防止される.
       (See: JvmtiEnvBase::is_valid()))
      ---------------------------------------- -}

	  // Mark as invalid.
	  _magic = DISPOSED_MAGIC;
	
  {- -------------------------------------------
  (1) 全ての capability を放棄する
      ---------------------------------------- -}

	  // Relinquish all capabilities.
	  jvmtiCapabilities *caps = get_capabilities();
	  JvmtiManageCapabilities::relinquish_capabilities(caps, caps, caps);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Same situation as with events (see above)
	  set_native_method_prefixes(0, NULL);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	#ifndef JVMTI_KERNEL
	  JvmtiTagMap* tag_map_to_deallocate = _tag_map;
	  set_tag_map(NULL);
	  // A tag map can be big, deallocate it now
	  if (tag_map_to_deallocate != NULL) {
	    delete tag_map_to_deallocate;
	  }
	#endif // !JVMTI_KERNEL
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  _needs_clean_up = true;
	}
	
```


