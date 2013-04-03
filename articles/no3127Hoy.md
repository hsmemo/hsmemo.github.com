---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1RemSet.inline.hpp

### 名前(function name)
```
template <class T>
inline void UpdateRSOrPushRefOopClosure::do_oop_work(T* p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  oop obj = oopDesc::load_decode_heap_oop(p);

  {- -------------------------------------------
  (1) (ASSERT)
      ---------------------------------------- -}

	#ifdef ASSERT
	  // can't do because of races
	  // assert(obj == NULL || obj->is_oop(), "expected an oop");
	
	  // Do the safe subset of is_oop
	  if (obj != NULL) {
	#ifdef CHECK_UNHANDLED_OOPS
	    oopDesc* o = obj.obj();
	#else
	    oopDesc* o = obj;
	#endif // CHECK_UNHANDLED_OOPS
	    assert((intptr_t)o % MinObjAlignmentInBytes == 0, "not oop aligned");
	    assert(Universe::heap()->is_in_reserved(obj), "must be in heap");
	  }
	#endif // ASSERT
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_from != NULL, "from region must be non-NULL");
	
  {- -------------------------------------------
  (1) (処理対象のオブジェクトに対応する HeapRegion が見つからなければ何もしない.
       また, 参照元と参照先が同じ HeapRegion だった場合も何もしない.)
      ---------------------------------------- -}

	  HeapRegion* to = _g1->heap_region_containing(obj);
	  if (to != NULL && _from != to) {

  {- -------------------------------------------
  (1) _record_refs_into_cset フラグが true で, かつポインタが collection set 内を差していれば (?? #TODO), 
      _push_ref_cl フィールドの...に対して do_oop() を呼び出す.
      (<= ここで _push_ref_cl フィールドに入っているのは G1ParPushHeapRSClosure??)
      そうでない場合は, G1RemSet::par_write_ref() を呼び出す (なお, この呼び出し時には worker 番号も指定).
  
      (なお, _record_refs_into_cset フラグが true になるのは, evacuation pause 中で RSet update を行っている間だけ.
       また, この場合でも, 既に self_forwarded されているなら処理はしない.)
      ---------------------------------------- -}

	    // The _record_refs_into_cset flag is true during the RSet
	    // updating part of an evacuation pause. It is false at all
	    // other times:
	    //  * rebuilding the rembered sets after a full GC
	    //  * during concurrent refinement.
	    //  * updating the remembered sets of regions in the collection
	    //    set in the event of an evacuation failure (when deferred
	    //    updates are enabled).
	
	    if (_record_refs_into_cset && to->in_collection_set()) {
	      // We are recording references that point into the collection
	      // set and this particular reference does exactly that...
	      // If the referenced object has already been forwarded
	      // to itself, we are handling an evacuation failure and
	      // we have already visited/tried to copy this object
	      // there is no need to retry.
	      if (!self_forwarded(obj)) {
	        assert(_push_ref_cl != NULL, "should not be null");
	        // Push the reference in the refs queue of the G1ParScanThreadState
	        // instance for this worker thread.
	        _push_ref_cl->do_oop(p);
	      }
	
	      // Deferred updates to the CSet are either discarded (in the normal case),
	      // or processed (if an evacuation failure occurs) at the end
	      // of the collection.
	      // See G1RemSet::cleanup_after_oops_into_collection_set_do().
	    } else {
	      // We either don't care about pushing references that point into the
	      // collection set (i.e. we're not during an evacuation pause) _or_
	      // the reference doesn't point into the collection set. Either way
	      // we add the reference directly to the RSet of the region containing
	      // the referenced object.
	      _g1_rem_set->par_write_ref(_from, p, _worker_i);
	    }
	  }
	}
	
```


