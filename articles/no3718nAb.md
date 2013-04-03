---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/opto/graphKit.cpp
### 説明(description)

```
// vanilla/CMS post barrier
// Insert a write-barrier store.  This is to let generational GC work; we have
// to flag all oop-stores before the next GC point.
```

### 名前(function name)
```
void GraphKit::write_barrier_post(Node* oop_store,
                                  Node* obj,
                                  Node* adr,
                                  uint  adr_idx,
                                  Node* val,
                                  bool use_precise) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし書き込んだポインタ値が NULL であれば, card table に記録する必要は無いので, ここでリターン (この場合, ノードは生成しない).
  
      また, 書き込んだポインタ値が Perm 領域内を指しているのであれば
      (Old から New へのポインタにはならないので) ここでリターン (この場合, ノードは生成しない).
        (ただし, CMS の場合だけは書き込んだポインタが Perm を指していてもここでリターンしない (card table を変更するノードを生成する).
         See: can_elide_permanent_oop_store_barriers())
      ---------------------------------------- -}

	  // No store check needed if we're storing a NULL or an old object
	  // (latter case is probably a string constant). The concurrent
	  // mark sweep garbage collector, however, needs to have all nonNull
	  // oop updates flagged via card-marks.
	  if (val != NULL && val->is_Con()) {
	    // must be either an oop or NULL
	    const Type* t = val->bottom_type();
	    if (t == TypePtr::NULL_PTR || t == Type::TOP)
	      // stores of null never (?) need barriers
	      return;
	    ciObject* con = t->is_oopptr()->const_oop();
	    if (con != NULL
	        && con->is_perm()
	        && Universe::heap()->can_elide_permanent_oop_store_barriers())
	      // no store barrier needed, because no old-to-new ref created
	      return;
	  }
	
  {- -------------------------------------------
  (1) もし書き込み先のオブジェクトが生成された直後のオブジェクト (Eden 領域内にある) であれば, 
      (Old から New へのポインタにはならないので) ここでリターン (この場合, ノードは生成しない).
      
      (なお, ...#TODO)
      ---------------------------------------- -}

	  if (use_ReduceInitialCardMarks()
	      && obj == just_allocated_object(control())) {
	    // We can skip marks on a freshly-allocated object in Eden.
	    // Keep this code in sync with new_store_pre_barrier() in runtime.cpp.
	    // That routine informs GC to take appropriate compensating steps,
	    // upon a slow-path allocation, so as to make this card-mark
	    // elision safe.
	    return;
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (!use_precise) {
	    // All card marks for a (non-array) instance are in one place:
	    adr = obj;
	  }
	  // (Else it's an array (or unknown), and we want more precise card marks.)

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(adr != NULL, "");
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  IdealKit ideal(this, true);
	
  {- -------------------------------------------
  (1) 書き込んだアドレスに対応する card table 内のエントリのアドレスを計算する.
      (アドレスを CastPX ノードで整数値にキャストした後 
       URShiftX ノードで CardTableModRefBS::card_shift 分だけ右シフトし, 
       AddP ノードで card table の先頭アドレスに足す.)
      ---------------------------------------- -}

	  // Convert the pointer to an int prior to doing math on it
	  Node* cast = __ CastPX(__ ctrl(), adr);
	
	  // Divide by card size
	  assert(Universe::heap()->barrier_set()->kind() == BarrierSet::CardTableModRef,
	         "Only one we handle so far.");
	  Node* card_offset = __ URShiftX( cast, __ ConI(CardTableModRefBS::card_shift) );
	
	  // Combine card table base and card offset
	  Node* card_adr = __ AddP(__ top(), byte_map_base_node(), card_offset );
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Get the alias_index for raw card-mark memory
	  int adr_type = Compile::AliasIdxRaw;
	  Node*   zero = __ ConI(0); // Dirty card value
	  BasicType bt = T_BYTE;
	
  {- -------------------------------------------
  (1) card table 内の該当エントリを dirty 化するためのノードを生成する.
    
      (CMS でない場合には store() で, CMS の場合には storeCM() でノードを生成)
      (なお, 0 が dirty を示す (See: dirty_card))
      (なお, UseCondCardMark オプションが指定されている場合は, 
       まず card の値を確認し, 既に dirty 化済みであれば書き込まないようにしている.)
      ---------------------------------------- -}

	  if (UseCondCardMark) {
	    // The classic GC reference write barrier is typically implemented
	    // as a store into the global card mark table.  Unfortunately
	    // unconditional stores can result in false sharing and excessive
	    // coherence traffic as well as false transactional aborts.
	    // UseCondCardMark enables MP "polite" conditional card mark
	    // stores.  In theory we could relax the load from ctrl() to
	    // no_ctrl, but that doesn't buy much latitude.
	    Node* card_val = __ load( __ ctrl(), card_adr, TypeInt::BYTE, bt, adr_type);
	    __ if_then(card_val, BoolTest::ne, zero);
	  }
	
	  // Smash zero into card
	  if( !UseConcMarkSweepGC ) {
	    __ store(__ ctrl(), card_adr, zero, bt, adr_type);
	  } else {
	    // Specialized path for CM store barrier
	    __ storeCM(__ ctrl(), card_adr, zero, oop_store, adr_idx, bt, adr_type);
	  }
	
	  if (UseCondCardMark) {
	    __ end_if();
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Final sync IdealKit and GraphKit.
	  final_sync(ideal);
	}
	
```


