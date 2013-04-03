---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.cpp
### 説明(description)

```
// We get passed the space_top value to prevent us from traversing into
// the old_gen promotion labs, which cannot be safely parsed.
```

### 名前(function name)
```
void CardTableExtension::scavenge_contents(ObjectStartArray* start_array,
                                           MutableSpace* sp,
                                           HeapWord* space_top,
                                           PSPromotionManager* pm)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(start_array != NULL && sp != NULL && pm != NULL, "Sanity");
	  assert(start_array->covered_region().contains(sp->used_region()),
	         "ObjectStartArray does not cover space");
	
  {- -------------------------------------------
  (1) (もし処理対象の領域 (現状だと PSPermGen しか指定されないが..) が空であれば何もせずにいきなり終了.
       空でない場合には, 以下のブロック内の処理を行う.)
      ---------------------------------------- -}

	  if (sp->not_empty()) {

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    oop* sp_top = (oop*)space_top;
	    oop* prev_top = NULL;
	    jbyte* current_card = byte_for(sp->bottom());
	    jbyte* end_card     = byte_for(sp_top - 1);    // sp_top is exclusive

  {- -------------------------------------------
  (1) 以下の while ループ内で, 指定された領域(現状だと PSPermGen しか指定されないが..)の
      bottom から top (上記の変数宣言では end_card) までを処理する.
  
      一回のループにつき, 「連続した非cleanなcard領域」一つ分(直近の非cleanなcardからその次のcleanなcardまで)を処理していく.
      ただし, 処理対象の領域の最後に大きなオブジェクトがあって
      それがその後ろの clean な card をまるまる占有しているような状態だと少し面倒なこと(後述)になるので, 
      その場合は最後のオブジェクトが占めている領域は全て処理対象に含めることとする.
      ---------------------------------------- -}

	    // scan card marking array
	    while (current_card <= end_card) {
	      jbyte value = *current_card;
	      // skip clean cards

  {- -------------------------------------------
  (1) 現在見ていた card が clean だった場合には, 
      することはないので処理対象を次のカードに変更してループの次の週へ.
      (要は clean な card の部分をスキップしているだけ)
      ---------------------------------------- -}

	      if (card_is_clean(value)) {
	        current_card++;

  {- -------------------------------------------
  (1) 現在見ていた card が clean でなければ (= 非cleanな card が見つかったら), 以下の else のブロック内の処理を行う.
      ---------------------------------------- -}

	      } else {

    {- -------------------------------------------
  (1.1) 今回のループでの処理範囲の開始アドレス(以下の bottom_obj)を計算する.
    
        (?? 最後の文は bottom じゃなくて bottom_obj では?? 
           if (bottom < prev_top) bottom = prev_top;
  
            bottom はこの後で使われていないので変更しても無意味なような...)
        ---------------------------------------- -}

	        // we found a non-clean card
	        jbyte* first_nonclean_card = current_card++;
	        oop* bottom = (oop*)addr_for(first_nonclean_card);
	        // find object starting on card
	        oop* bottom_obj = (oop*)start_array->object_start((HeapWord*)bottom);
	        // bottom_obj = (oop*)start_array->object_start((HeapWord*)bottom);
	        assert(bottom_obj <= bottom, "just checking");
	        // make sure we don't scan oops we already looked at
	        if (bottom < prev_top) bottom = prev_top;

    {- -------------------------------------------
  (1.1) 今回のループでの処理範囲の終端(以下の top)を計算する.
    
        もし複数の card にまたがるような大きなオブジェクトが有り, そのオブジェクト内に複数箇所 clean でない箇所があれば
        同じオブジェクトを2回チェックしてしまう恐れがあるため, それもここで確認している模様.
  
        (処理としては, 非clean な card が続く限りスキップし, 次の clean な card を見つける.
         そして, 見つかった最後の 非clean card 中の最後のオブジェクトを調べてみる.
         そのオブジェクトの大きさが非常に大きくて次の card も完全に含んでしまうようであれば, 
         今回の処理対象をそのオブジェクトの終端まで拡張して, そのオブジェクトに含まれる後ろの card もまとめて処理してしまうことにする.
         
         拡張したら, 拡張先がまた 非clean だった場合は同じことが起こるかもしれないので, 同様の処理を繰り返し, 
         拡張先が clean になるか終端に達するまで続ける.)
        ---------------------------------------- -}

	        // figure out when to stop scanning
	        jbyte* first_clean_card;
	        oop* top;
	        bool restart_scanning;
	        do {
	          restart_scanning = false;
	          // find a clean card
	          while (current_card <= end_card) {
	            value = *current_card;
	            if (card_is_clean(value)) break;
	            current_card++;
	          }
	          // check if we reached the end, if so we are done
	          if (current_card >= end_card) {
	            first_clean_card = end_card + 1;
	            current_card++;
	            top = sp_top;
	          } else {
	            // we have a clean card, find object starting on that card
	            first_clean_card = current_card++;
	            top = (oop*)addr_for(first_clean_card);
	            oop* top_obj = (oop*)start_array->object_start((HeapWord*)top);
	            // top_obj = (oop*)start_array->object_start((HeapWord*)top);
	            assert(top_obj <= top, "just checking");
	            if (oop(top_obj)->is_objArray() || oop(top_obj)->is_typeArray()) {
	              // an arrayOop is starting on the clean card - since we do exact store
	              // checks for objArrays we are done
	            } else {
	              // otherwise, it is possible that the object starting on the clean card
	              // spans the entire card, and that the store happened on a later card.
	              // figure out where the object ends
	              top = top_obj + oop(top_obj)->size();
	              jbyte* top_card = CardTableModRefBS::byte_for(top - 1);   // top is exclusive
	              if (top_card > first_clean_card) {
	                // object ends a different card
	                current_card = top_card + 1;
	                if (card_is_clean(*top_card)) {
	                  // the ending card is clean, we are done
	                  first_clean_card = top_card;
	                } else {
	                  // the ending card is not clean, continue scanning at start of do-while
	                  restart_scanning = true;
	                }
	              } else {
	                // object ends on the clean card, we are done.
	                assert(first_clean_card == top_card, "just checking");
	              }
	            }
	          }
	        } while (restart_scanning);

    {- -------------------------------------------
  (1.1) 処理範囲の計算が終わった(= 対応する card table の値は不要になった)ので, card table の対応箇所を clean に戻しておく.
        ---------------------------------------- -}

	        // we know which cards to scan, now clear them
	        while (first_nonclean_card < first_clean_card) {
	          *first_nonclean_card++ = clean_card;
	        }

    {- -------------------------------------------
  (1.1) 範囲内(bottom_obj から top まで)のオブジェクト全てに対して oopDesc::push_contents() を呼び出すことで, 
        引数で渡された PSPromotionManager オブジェクトに
        該当範囲に含まれるオブジェクト内のポインタを登録する.
    
        (処理自体は, bottom_obj から初めてオブジェクトの size() 分だけ bottom_obj を増加させながら top までループする, という処理.)
        (なお, 登録量が多くなりすぎた場合に備えて
         PSPromotionManager::drain_stacks_cond_depth() も呼び出している.
         このため, 登録量が多い場合には, 
         この場で PSPromotionManager::drain_stacks_depth() 処理も行いながら処理が進められることになる.)
        ---------------------------------------- -}

	        // scan oops in objects
	        do {
	          oop(bottom_obj)->push_contents(pm);
	          bottom_obj += oop(bottom_obj)->size();
	          assert(bottom_obj <= sp_top, "just checking");
	        } while (bottom_obj < top);
	        pm->drain_stacks_cond_depth();

    {- -------------------------------------------
  (1.1) 一度処理したオブジェクトをまた処理してしまわないよう, prev_top 変数を更新しておく.
        ---------------------------------------- -}

	        // remember top oop* scanned
	        prev_top = top;
	      }
	    }
	  }
	}
	
```


