---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/jniHandles.cpp

### 名前(function name)
```
jobject JNIHandles::make_weak_global(Handle obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!Universe::heap()->is_gc_active(), "can't extend the root set during GC");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jobject res = NULL;

  {- -------------------------------------------
  (1) もし引数の oop が NULL でなければ, 
      JNIHandleBlock::allocate_handle() で JNIHandles::_weak_global_handles 内に格納する.
      ---------------------------------------- -}

	  if (!obj.is_null()) {
	    // ignore null handles
	    MutexLocker ml(JNIGlobalHandle_lock);
	    assert(Universe::heap()->is_in_reserved(obj()), "sanity check");
	    res = _weak_global_handles->allocate_handle(obj());

  {- -------------------------------------------
  (1) 引数の oop が NULL だった場合は, 通常時には何もしない.
      デバッグ時には次の処理が行われる.
  
      (デバッグ用の処理) (CHECK_UNHANDLED_OOPS_ONLY 時にのみ実行) (See: UnhandledOops)
      unhandled oop をクリアする (? #TODO)
      ---------------------------------------- -}

	  } else {
	    CHECK_UNHANDLED_OOPS_ONLY(Thread::current()->clear_unhandled_oops());
	  }

  {- -------------------------------------------
  (1) 結果をリターン.
      ---------------------------------------- -}

	  return res;
	}
	
```


