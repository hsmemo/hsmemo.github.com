---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.cpp

### 名前(function name)
```
oop PSPromotionManager::oop_promotion_failed(oop obj, markOop obj_mark) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_old_gen_is_full || PromotionFailureALot, "Sanity");
	
  {- -------------------------------------------
  (1) CAS で, コピー元のオブジェクトの mark フィールドに, そのオブジェクト自身へのポインタを埋め込む.
      (CAS で比較する値としては, 自分がそのオブジェクトの処理を開始した時点での mark 値とする.
       CAS が成功してしまったら, 他のスレッドは誰もそのオブジェクトの処理をしていないということになる.)
      ---------------------------------------- -}

	  // Attempt to CAS in the header.
	  // This tests if the header is still the same as when
	  // this started.  If it is the same (i.e., no forwarding
	  // pointer has been installed), then this thread owns
	  // it.
	  if (obj->cas_forward_to(obj, obj_mark)) {

  {- -------------------------------------------
  (1) CAS が成功した場合は, oopDesc::push_contents() で
      オブジェクト内のポインタを PSPromotionManager 内のタスクキューに追加する.
  
      その後, PSScavenge::oop_promotion_failed() で
      (必要であれば) 元の mark 値を PSScavenge::_preserved_mark_stack に保存しておく. 
        (forwarding pointer を埋めてしまったので, そのままだと元の mark 情報が失われるため)
      また, 確保が失敗したことが後から認識できるよう, _promotion_failed フィールドも変更しておく.
      ---------------------------------------- -}

	    // We won any races, we "own" this object.
	    assert(obj == obj->forwardee(), "Sanity");
	
	    obj->push_contents(this);
	
	    // Save the mark if needed
	    PSScavenge::oop_promotion_failed(obj, obj_mark);

  {- -------------------------------------------
  (1) CAS が失敗した場合は, 他の誰かがコピーしたということなので, 
      単にそのコピー先を取得するだけ.
      ---------------------------------------- -}

	  }  else {
	    // We lost, someone else "owns" this object
	    guarantee(obj->is_forwarded(), "Object must be forwarded if the cas failed.");
	
	    // No unallocation to worry about.
	    obj = obj->forwardee();
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#ifdef DEBUG
	  if (TraceScavenge) {
	    gclog_or_tty->print_cr("&#x7b;&#x25;s %s 0x%x (%d)}",
	                           "promotion-failure",
	                           obj->blueprint()->internal_name(),
	                           obj, obj->size());
	
	  }
	#endif
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return obj;
	}
	
```


