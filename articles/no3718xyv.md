---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/cardTableExtension.cpp

### 名前(function name)
```
void CardTableExtension::scavenge_contents_parallel(ObjectStartArray* start_array,
                                                    MutableSpace* sp,
                                                    HeapWord* space_top,
                                                    PSPromotionManager* pm,
                                                    uint stripe_number) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int ssize = 128; // Naked constant!  Work unit = 64k.
	  int dirty_card_count = 0;
	
	  oop* sp_top = (oop*)space_top;
	  jbyte* start_card = byte_for(sp->bottom());
	  jbyte* end_card   = byte_for(sp_top - 1) + 1;
	  oop* last_scanned = NULL; // Prevent scanning objects more than once

  {- -------------------------------------------
  (1) 以下のループ内で, 
      指定された領域(現状だと PSOldGen しか指定されないが..)の
      bottom (上記の変数宣言では start_card) から top (上記の変数宣言では end_card) までを処理する.
  
      ただし bottom から top までの全てを処理するわけではなく, これを ParallelGCThreads 個分に分けて並列処理する.
      (つまり, 1つの CardTableExtension::scavenge_contents_parallel() では, 
       全体の ParallelGCThreads 分の1 だけを処理すればいい).
    
      自分の担当分は stripe_number という引数で示されている. 具体的には以下のように処理する. (#TODO 絵でも描く?)
      (1) 先頭(start_card) + stripe_number*ssize のアドレスから ssize バイト分を処理
      (2) 次に, ssize*ParallelGCThreads 分だけ進み, また ssize バイト分を処理
          (つまり, 先頭 + stripe_number*ssize + ssize*ParallelGCThreads のアドレスから ssize バイト分)
      (3) さらに ssize*ParallelGCThreads 分だけ進み, ... (以下, 終端(end_card)に到達するまで繰り返し)
    
      (なお正確に言うと, 
       start_card と end_card は処理対象の先頭アドレスや終端アドレスそのものではなく, 
       そのアドレスに対応する CardTableExtension 内のエントリのアドレス.
       (dereference すると, そのアドレスが含まれる card の barrier set の値(jbyte 値)が得られる)
  
       ssize の 128 という値が 64K に対応するのは, 1 card の大きさが 512 byte であるため.
       (See: CardTableExtension))
      ---------------------------------------- -}

	  for (jbyte* slice = start_card; slice < end_card; slice += ssize*ParallelGCThreads) {

    {- -------------------------------------------
  (1.1) (各ループでの担当範囲は, 上記の通り
         「先頭 + stripe_number*ssize + ssize*ParallelGCThreads*ループ回数」のアドレスから ssize バイト分)
        ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) 担当範囲の開始アドレス(に対応するcard, 以下の worker_start_card)が終端(end_card)を超えてしまっていれば処理は終了.
        ---------------------------------------- -}

	    jbyte* worker_start_card = slice + stripe_number * ssize;
	    if (worker_start_card >= end_card)
	      return; // We're done.
	
    {- -------------------------------------------
  (1.1) 担当範囲の終了アドレス(に対応するcard, 以下の worker_end_card)が終端(end_card)を超えていたら, 
        終了アドレスの方を終端に切り詰めておく.
        ---------------------------------------- -}

	    jbyte* worker_end_card = worker_start_card + ssize;
	    if (worker_end_card > end_card)
	      worker_end_card = end_card;
	
    {- -------------------------------------------
  (1.1) (オブジェクトが担当範囲の境界をまたいでいる(= 複数の担当領域にまたがっている)場合は
         「オブジェクトの先頭」を含んでいる方が処理を行うこととする, とのこと)
        ---------------------------------------- -}

	    // We do not want to scan objects more than once. In order to accomplish
	    // this, we assert that any object with an object head inside our 'slice'
	    // belongs to us. We may need to extend the range of scanned cards if the
	    // last object continues into the next 'slice'.
	    //
	    // Note! ending cards are exclusive!

    {- -------------------------------------------
  (1.1) (変数宣言など)
        (それぞれ担当範囲の開始/終端アドレス)
        ---------------------------------------- -}

	    HeapWord* slice_start = addr_for(worker_start_card);
	    HeapWord* slice_end = MIN2((HeapWord*) sp_top, addr_for(worker_end_card));
	
    {- -------------------------------------------
  (1.1) もし自分の担当範囲内に1つも「オブジェクトの先頭」がなければ
        (= 前の担当範囲内で始まったオブジェクトが非常に巨大で, 自分の担当範囲全てが埋め尽くされているケース)
        処理するオブジェクトがいないので continue でループの次の周の処理に移る.
        ---------------------------------------- -}

	    // If there are not objects starting within the chunk, skip it.
	    if (!start_array->object_starts_in_range(slice_start, slice_end)) {
	      continue;
	    }

    {- -------------------------------------------
  (1.1) 担当範囲の開始アドレス(に対応するcard, 以下の worker_start_card)を, 
        担当範囲に存在する最初のオブジェクトの開始アドレス(に対応するcard)に変更しておく.
        ---------------------------------------- -}

	    // Update our beginning addr
	    HeapWord* first_object = start_array->object_start(slice_start);
	    debug_only(oop* first_object_within_slice = (oop*) first_object;)
	    if (first_object < slice_start) {
	      last_scanned = (oop*)(first_object + oop(first_object)->size());
	      debug_only(first_object_within_slice = last_scanned;)
	      worker_start_card = byte_for(last_scanned);
	    }
	
    {- -------------------------------------------
  (1.1) 担当範囲の終了アドレス(に対応するcard, 以下の worker_end_card)を, 
        担当範囲に存在する最後のオブジェクトの終端アドレス(に対応するcard)に変更しておく.
        (ただし, 処理対象のヒープの終端(end_card)を超えてしまったら, 超えないように切り詰めておく)
        ---------------------------------------- -}

	    // Update the ending addr
	    if (slice_end < (HeapWord*)sp_top) {
	      // The subtraction is important! An object may start precisely at slice_end.
	      HeapWord* last_object = start_array->object_start(slice_end - 1);
	      slice_end = last_object + oop(last_object)->size();
	      // worker_end_card is exclusive, so bump it one past the end of last_object's
	      // covered span.
	      worker_end_card = byte_for(slice_end) + 1;
	
	      if (worker_end_card > end_card)
	        worker_end_card = end_card;
	    }
	
    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	    assert(slice_end <= (HeapWord*)sp_top, "Last object in slice crosses space boundary");
	    assert(is_valid_card_address(worker_start_card), "Invalid worker start card");
	    assert(is_valid_card_address(worker_end_card), "Invalid worker end card");
	    // Note that worker_start_card >= worker_end_card is legal, and happens when
	    // an object spans an entire slice.
	    assert(worker_start_card <= end_card, "worker start card beyond end card");
	    assert(worker_end_card <= end_card, "worker end card beyond end card");
	
    {- -------------------------------------------
  (1.1) (以下の while ループで, 担当範囲の先頭(worker_start_card)から終わり(worker_end_card)まで処理を行っていく)
    
        一回のループにつき, 「連続した非cleanなcard領域」一つ分(直近の非cleanなcardからその次のcleanなcardまで)を処理していく.
        ただし, 処理対象の領域の最後に大きなオブジェクトがあって
        それがその後ろの clean な card をまるまる占有しているような状態だと少し面倒なこと(後述)になるので, 
        その場合は最後のオブジェクトが占めている領域は全て処理対象に含めることとする.
        ---------------------------------------- -}

	    jbyte* current_card = worker_start_card;
	    while (current_card < worker_end_card) {

      {- -------------------------------------------
  (1.1.1) まず, 処理対象の開始card (以下の first_unclean_card) を見つける.
          (処理としては clean な card の部分をスキップしていって, 非cleanな card を見つけるだけ)
          ---------------------------------------- -}

	      // Find an unclean card.
	      while (current_card < worker_end_card && card_is_clean(*current_card)) {
	        current_card++;
	      }
	      jbyte* first_unclean_card = current_card;
	
      {- -------------------------------------------
  (1.1.1) 次に処理対象の終端となるcard(以下の following_clean_card)を見つける.
          
          もし複数の card にまたがるような大きなオブジェクトが有り, そのオブジェクト内に複数箇所 clean でない箇所があれば
          同じオブジェクトを2回チェックしてしまう恐れがあるため, それもここで確認している.
          (そのために last_scanned という変数を用意しているのでは? という話もあるが
           これだけだと適切に card marking するのがちょっと面倒なのでこうしているとのこと.
           (New 領域を指している card を youngergen_card にする処理のことだと思われる.
            確かに, 同じ領域が2回処理対象になると, last_scanned でスキップしていいと分かったとしても, 
            2回目は card を clean していいのかそのままにした方がいいのかの判断が難しそうか.))
  
          (処理としては, 非clean な card が続く限りスキップし, 次の clean な card を見つける.
           そして, 見つかった最後の 非clean card 中の最後のオブジェクトを調べてみる.
           そのオブジェクトが非常に大きく, 次の card も完全に含んでしまうようであれば, 
           今回の処理対象をそのオブジェクトの終端まで拡張して, そのオブジェクトに含まれる後ろの card もまとめて処理してしまうことにする.
           
           拡張したら, 拡張先がまた 非clean だった場合は同じことが起こるかもしれないので, 同様の処理を繰り返し, 
           拡張先が clean になるか終端に達するまで続ける.)
          ---------------------------------------- -}

	      // Find the end of a run of contiguous unclean cards
	      while (current_card < worker_end_card && !card_is_clean(*current_card)) {
	        while (current_card < worker_end_card && !card_is_clean(*current_card)) {
	          current_card++;
	        }
	
	        if (current_card < worker_end_card) {
	          // Some objects may be large enough to span several cards. If such
	          // an object has more than one dirty card, separated by a clean card,
	          // we will attempt to scan it twice. The test against "last_scanned"
	          // prevents the redundant object scan, but it does not prevent newly
	          // marked cards from being cleaned.
	          HeapWord* last_object_in_dirty_region = start_array->object_start(addr_for(current_card)-1);
	          size_t size_of_last_object = oop(last_object_in_dirty_region)->size();
	          HeapWord* end_of_last_object = last_object_in_dirty_region + size_of_last_object;
	          jbyte* ending_card_of_last_object = byte_for(end_of_last_object);
	          assert(ending_card_of_last_object <= worker_end_card, "ending_card_of_last_object is greater than worker_end_card");
	          if (ending_card_of_last_object > current_card) {
	            // This means the object spans the next complete card.
	            // We need to bump the current_card to ending_card_of_last_object
	            current_card = ending_card_of_last_object;
	          }
	        }
	      }
	      jbyte* following_clean_card = current_card;
	
      {- -------------------------------------------
  (1.1.1) 上の処理で clean ではない card が見つかっていれば, 以下の if のブロック内で処理を行う.
          ---------------------------------------- -}

	      if (first_unclean_card < worker_end_card) {

        {- -------------------------------------------
  (1.1.1.1) first_unclean_card を基に, 今回のループでの処理範囲の開始アドレス(以下の p)を計算する.
            (なお, コメントによると p >= last_scanned でないと同じオブジェクトを複数回スキャンしてしまうためチェックを入れているとのこと.
             チェックを削りたいなら, last_scanned が first_object_within_slice のケースを考慮するようにとのこと.)
            ---------------------------------------- -}

	        oop* p = (oop*) start_array->object_start(addr_for(first_unclean_card));
	        assert((HeapWord*)p <= addr_for(first_unclean_card), "checking");
	        // "p" should always be >= "last_scanned" because newly GC dirtied
	        // cards are no longer scanned again (see comment at end
	        // of loop on the increment of "current_card").  Test that
	        // hypothesis before removing this code.
	        // If this code is removed, deal with the first time through
	        // the loop when the last_scanned is the object starting in
	        // the previous slice.
	        assert((p >= last_scanned) ||
	               (last_scanned == first_object_within_slice),
	               "Should no longer be possible");
	        if (p < last_scanned) {
	          // Avoid scanning more than once; this can happen because
	          // newgen cards set by GC may a different set than the
	          // originally dirty set
	          p = last_scanned;
	        }

        {- -------------------------------------------
  (1.1.1.1) following_clean_card を基に, 今回のループでの処理範囲の終端(以下の to)を計算する.
            ---------------------------------------- -}

	        oop* to = (oop*)addr_for(following_clean_card);
	
	        // Test slice_end first!
	        if ((HeapWord*)to > slice_end) {
	          to = (oop*)slice_end;
	        } else if (to > sp_top) {
	          to = sp_top;
	        }
	
        {- -------------------------------------------
  (1.1.1.1) 処理範囲の計算が終わった(= 対応する card table の値は不要になった)ので, card table の対応箇所を clean に戻しておく.
            ---------------------------------------- -}

	        // we know which cards to scan, now clear them
	        if (first_unclean_card <= worker_start_card+1)
	          first_unclean_card = worker_start_card+1;
	        if (following_clean_card >= worker_end_card-1)
	          following_clean_card = worker_end_card-1;
	
	        while (first_unclean_card < following_clean_card) {
	          *first_unclean_card++ = clean_card;
	        }
	
        {- -------------------------------------------
  (1.1.1.1) 範囲内(p から to まで)のオブジェクト全てに対して oopDesc::push_contents() を呼び出すことで, 
            引数で渡された PSPromotionManager オブジェクトに
            該当範囲に含まれるオブジェクト内のポインタを登録する.
    
            (処理自体は, p から初めてオブジェクトの size() 分だけ p を増加させながら to までループする, という処理.)
            (なお PrefetchScanIntervalInBytes オプションの値が 0 でない場合には, プリフェッチしながら処理を行う模様.)
            (なお, 登録量が多くなりすぎた場合に備えて
             PSPromotionManager::drain_stacks_cond_depth() も呼び出している.
             このため, 登録量が多い場合には, 
             この場で PSPromotionManager::drain_stacks_depth() 処理も行いながら処理が進められることになる.)
            ---------------------------------------- -}

	        const int interval = PrefetchScanIntervalInBytes;
	        // scan all objects in the range
	        if (interval != 0) {
	          while (p < to) {
	            Prefetch::write(p, interval);
	            oop m = oop(p);
	            assert(m->is_oop_or_null(), "check for header");
	            m->push_contents(pm);
	            p += m->size();
	          }
	          pm->drain_stacks_cond_depth();
	        } else {
	          while (p < to) {
	            oop m = oop(p);
	            assert(m->is_oop_or_null(), "check for header");
	            m->push_contents(pm);
	            p += m->size();
	          }
	          pm->drain_stacks_cond_depth();
	        }
	        last_scanned = p;
	      }

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	      // "current_card" is still the "following_clean_card" or
	      // the current_card is >= the worker_end_card so the
	      // loop will not execute again.
	      assert((current_card == following_clean_card) ||
	             (current_card >= worker_end_card),
	        "current_card should only be incremented if it still equals "
	        "following_clean_card");

    {- -------------------------------------------
  (1.1) 処理対象のcard(current_card)を一つインクリメントしてループの次の周へ.
        ---------------------------------------- -}

	      // Increment current_card so that it is not processed again.
	      // It may now be dirty because a old-to-young pointer was
	      // found on it an updated.  If it is now dirty, it cannot be
	      // be safely cleaned in the next iteration.
	      current_card++;
	    }
	  }
	}
	
```


