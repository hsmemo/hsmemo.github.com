---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp
### 説明(description)

```
// This method is called whenever an attempt to promote an object
// fails. Some markOops will need preservation, some will not. Note
// that the entire eden is traversed after a failed promotion, with
// all forwarded headers replaced by the default markOop. This means
// it is not neccessary to preserve most markOops.
```

### 名前(function name)
```
void PSScavenge::oop_promotion_failed(oop obj, markOop obj_mark) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 失敗したことが後から認識できるよう, _promotion_failed フィールドを変更しておく.
      (確保失敗のケースでは, 後で PSScavenge::clean_up_failed_promotion() により後始末が行われる.
       See: PSScavenge::clean_up_failed_promotion())
      ---------------------------------------- -}

	  _promotion_failed = true;

  {- -------------------------------------------
  (1) もし元の mark フィールドの値を保存しておく必要があれば, 
      PSScavenge::_preserved_mark_stack と PSScavenge::_preserved_oop_stack に
      処理対象のオブジェクトとその元の mark フィールド値を記録しておく.
  
      (なお, この記録処理は ThreadCritical で排他した状態で行っている)
      ---------------------------------------- -}

	  if (obj_mark->must_be_preserved_for_promotion_failure(obj)) {
	    // Should use per-worker private stakcs hetre rather than
	    // locking a common pair of stacks.
	    ThreadCritical tc;
	    _preserved_oop_stack.push(obj);
	    _preserved_mark_stack.push(obj_mark);
	  }
	}
	
```


