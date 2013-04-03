---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp

### 名前(function name)
```
void
G1CollectedHeap::heap_region_par_iterate_chunked(HeapRegionClosure* cl,
                                                 int worker,
                                                 jint claim_value) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      (start_index は, worker 引数に応じて変わる値. 各 GangWorker 毎の開始位置を示す)
      ---------------------------------------- -}

	  const size_t regions = n_regions();
	  const size_t worker_num = (G1CollectedHeap::use_parallel_gc_threads() ? ParallelGCThreads : 1);
	  // try to spread out the starting points of the workers
	  const size_t start_index = regions / worker_num * (size_t) worker;
	
  {- -------------------------------------------
  (1) (以下の for ループ内で HeapRegion を辿り, cl 引数で与えられた HeapRegionClosure を適用していく.
  
       ループでは, start_index の位置から開始して, 全ての HeapRegion を辿る.
       (最後の HeapRegion までたどり着いたら最初の HeapRegion に戻り, 全 HeapRegion を辿るまで続ける).
       そして, 各 HeapRegion に対して HeapRegion::claimHeapRegion() を呼び出して, その処理を行うことを宣言する. 
       HeapRegion::claimHeapRegion() が true を返せば, 
       カレントスレッドがその HeapRegion の担当者になったということなので, 処理を行う.
       逆に false であれば, 別のスレッドが担当しているということなので, 何もせずに次の HeapRegion に進む)
      ---------------------------------------- -}

	  // each worker will actually look at all regions
	  for (size_t count = 0; count < regions; ++count) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    const size_t index = (start_index + count) % regions;
	    assert(0 <= index && index < regions, "sanity");

    {- -------------------------------------------
  (1.1) 次の HeapRegion を取得する.
        ---------------------------------------- -}

	    HeapRegion* r = region_at(index);

    {- -------------------------------------------
  (1.1) 既に処理済みの HeapRegion (= claim_value() の値が claim_value 引数に等しいもの)や
        Humongous オブジェクトの途中に当たる HeapRegion であれば, 
        無視して次の HeapRegion へ進む.
        ---------------------------------------- -}

	    // we'll ignore "continues humongous" regions (we'll process them
	    // when we come across their corresponding "start humongous"
	    // region) and regions already claimed
	    if (r->claim_value() == claim_value || r->continuesHumongous()) {
	      continue;
	    }

    {- -------------------------------------------
  (1.1) HeapRegion::claimHeapRegion() を呼び出して, その HeapRegion の確保を試みる.
        失敗したら, この HeapRegion は無視して, 次の HeapRegion へ進む.
        ---------------------------------------- -}

	    // OK, try to claim it
	    if (r->claimHeapRegion(claim_value)) {

    {- -------------------------------------------
  (1.1) (以下は HeapRegion::claimHeapRegion() が成功した場合のパス)
        ---------------------------------------- -}

	      // success!

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	      assert(!r->continuesHumongous(), "sanity");

    {- -------------------------------------------
  (1.1) もし対象の HeapRegion が Humongous オブジェクトの先頭に当たる場合, 
        以下の if ブロック内で "continues humongous" を処理しておく.
    
        (コメントによると, 
         "continues humongous" に HeapRegionClosure を適用するのは
         "starts humongous" よりも先でないといけない, とのこと.
         これは, "starts humongous" に対して呼び出された際に 
         "continues humongous" を消去する closure が存在するため.
         (こうなると後続は "continues humongous" ではなくなるため再度処理対象になり二重に処理が走ることになる).
         ただし, そういった closure は一部だけで
         ほとんどの closure は "continues humongous" は単に無視するだけ, とのこと)
    
        (if ブロック内のループは, 対象の HeapRegion から開始し, 
         既に処理済みの HeapRegion (= claim_value() の値が claim_value 引数に等しいもの) か
         "continues humongous" ではない HeapRegion (= continuesHumongous() が false) に
         たどり着くまで繰り返す)
    
        (なお, "continues humongous" を処理する前に, 
         念のため HeapRegion::claimHeapRegion() で確保しているが, 
         失敗した場合は異常終了するだけ)
        ---------------------------------------- -}

	      if (r->startsHumongous()) {
	        // If the region is "starts humongous" we'll iterate over its
	        // "continues humongous" first; in fact we'll do them
	        // first. The order is important. In on case, calling the
	        // closure on the "starts humongous" region might de-allocate
	        // and clear all its "continues humongous" regions and, as a
	        // result, we might end up processing them twice. So, we'll do
	        // them first (notice: most closures will ignore them anyway) and
	        // then we'll do the "starts humongous" region.
	        for (size_t ch_index = index + 1; ch_index < regions; ++ch_index) {
	          HeapRegion* chr = region_at(ch_index);
	
	          // if the region has already been claimed or it's not
	          // "continues humongous" we're done
	          if (chr->claim_value() == claim_value ||
	              !chr->continuesHumongous()) {
	            break;
	          }
	
	          // Noone should have claimed it directly. We can given
	          // that we claimed its "starts humongous" region.
	          assert(chr->claim_value() != claim_value, "sanity");
	          assert(chr->humongous_start_region() == r, "sanity");
	
	          if (chr->claimHeapRegion(claim_value)) {
	            // we should always be able to claim it; noone else should
	            // be trying to claim this region
	
	            bool res2 = cl->doHeapRegion(chr);
	            assert(!res2, "Should not abort");
	
	            // Right now, this holds (i.e., no closure that actually
	            // does something with "continues humongous" regions
	            // clears them). We might have to weaken it in the future,
	            // but let's leave these two asserts here for extra safety.
	            assert(chr->continuesHumongous(), "should still be the case");
	            assert(chr->humongous_start_region() == r, "sanity");
	          } else {
	            guarantee(false, "we should not reach here");
	          }
	        }
	      }
	
    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	      assert(!r->continuesHumongous(), "sanity");

    {- -------------------------------------------
  (1.1) cl 引数で与えられた HeapRegionClosure に対して 
        HeapRegionClosure::doHeapRegion() を呼び出し, 
        対象の HeapRegion の処理を行う.
        ---------------------------------------- -}

	      bool res = cl->doHeapRegion(r);

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	      assert(!res, "Should not abort");
	    }
	  }
	}
	
```


