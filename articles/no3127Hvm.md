---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.cpp

### 名前(function name)
```
PosParPRT* OtherRegionsTable::delete_region_table() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	#if DRT_CENSUS
	  int histo[HistoSize] = { 0, 0, 0, 0, 0, 0 };
	  const int histo_limits[] = { 1, 4, 16, 64, 256, 2048 };
	#endif
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_m.owned_by_self(), "Precondition");
	  assert(_n_fine_entries == _max_fine_entries, "Precondition");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  PosParPRT* max = NULL;
	  jint max_occ = 0;
	  PosParPRT** max_prev;
	  size_t max_ind;
	
  {- -------------------------------------------
  (1) (以下のコードで, まず coarse map に追い出す対象を選択する.
       処理後には max 変数が追い出し対象をさすようになる.
  
       なお, SAMPLE_FOR_EVICTION の値に応じて 2通りの実装が書かれている.
       SAMPLE_FOR_EVICTION が 1 の場合は, いくつかのバケット内だけを見て追い出し対象を決める.
       そうで無い場合は, すべての PosParPRT を見て追い出し対象を決める.
  
       現状では, SAMPLE_FOR_EVICTION は 1 に #define されている)
      ---------------------------------------- -}

	#if SAMPLE_FOR_EVICTION

  {- -------------------------------------------
  (1) (以下が SAMPLE_FOR_EVICTION が 1 の場合のコード)
      ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) _max_fine_entries 内のいくつかの PosParPRT を辿り,
        調べた中でもっとも occupied が大きいものを追い出し対象として選択する.
  
        この処理パスでは _fine_eviction_sample_size で指定された個数のバケットしか調べない
        (なお, _fine_eviction_sample_size の値は初期化時に決まる).
  
        具体的な処理の流れは以下の通り.
  
        1. まずは _fine_eviction_start が指すバケットから調査を開始する.
           (ただしそのバケットが空であれば, バケットの番号が大きくなる方向に順に調べていき,
           一番最初に見つかった空ではないバケットを使う.
           当然だが, 終端まで行ったら 0 から折り返す. なお, 全部空ということはないはず(該当個所の assert 参照))
        2. そのバケット内の PosParPRT をすべて辿り occupied を調べる.
        3. 次のバケットに進む.
           次のバケットは, 現在のバケットから _fine_eviction_stride 分だけ番号が大きいものを選択する 
           (ただし, 当然だが _n_fine_entries (この時点ではバケット数(_max_fine_entries)と同じはず. 上のassert参照)を法として折り返す)
           (なお, _fine_eviction_stride は初期化時に決まる定数.
           バケット数を _fine_eviction_sample_size で割ったもの.
           なので, バケットが NULL でなければ, 等間隔で _fine_eviction_sample_size 回だけ調べる感じになる)
        ---------------------------------------- -}

	  size_t i = _fine_eviction_start;
	  for (size_t k = 0; k < _fine_eviction_sample_size; k++) {
	    size_t ii = i;
	    // Make sure we get a non-NULL sample.
	    while (_fine_grain_regions[ii] == NULL) {
	      ii++;
	      if (ii == _max_fine_entries) ii = 0;
	      guarantee(ii != i, "We must find one.");
	    }
	    PosParPRT** prev = &_fine_grain_regions[ii];
	    PosParPRT* cur = *prev;
	    while (cur != NULL) {
	      jint cur_occ = cur->occupied();
	      if (max == NULL || cur_occ > max_occ) {
	        max = cur;
	        max_prev = prev;
	        max_ind = i;
	        max_occ = cur_occ;
	      }
	      prev = cur->next_addr();
	      cur = cur->next();
	    }
	    i = i + _fine_eviction_stride;
	    if (i >= _n_fine_entries) i = i - _n_fine_entries;
	  }
	  _fine_eviction_start++;
	  if (_fine_eviction_start >= _n_fine_entries)
	    _fine_eviction_start -= _n_fine_entries;
	#else

  {- -------------------------------------------
  (1) (以下が SAMPLE_FOR_EVICTION が 1 ではない場合のコード)
      ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) _max_fine_entries 内のすべての PosParPRT を辿り,
        もっとも occupied が大きいものを追い出し対象として選択する.
        (open hash なので, 全バケットに対するループと
        バケット内の線形リストに対するループの二重ループ)
  
        (なお, DRT_CENSUS が 1 の場合は,
        occupied が histo_limits 配列中のどれかの数字よりも小さい PosParPRT が見つかった時点で脱出する.
        その際, 対応する histo や global_histo の箇所をインクリメントしている.
        ただし, 現状では DRT_CENSUS は 0.)
        ---------------------------------------- -}

	  for (int i = 0; i < _max_fine_entries; i++) {
	    PosParPRT** prev = &_fine_grain_regions[i];
	    PosParPRT* cur = *prev;
	    while (cur != NULL) {
	      jint cur_occ = cur->occupied();
	#if DRT_CENSUS
	      for (int k = 0; k < HistoSize; k++) {
	        if (cur_occ <= histo_limits[k]) {
	          histo[k]++; global_histo[k]++; break;
	        }
	      }
	#endif
	      if (max == NULL || cur_occ > max_occ) {
	        max = cur;
	        max_prev = prev;
	        max_ind = i;
	        max_occ = cur_occ;
	      }
	      prev = cur->next_addr();
	      cur = cur->next();
	    }
	  }
	#endif

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // XXX
	  guarantee(max != NULL, "Since _n_fine_entries > 0");

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#if DRT_CENSUS
	  gclog_or_tty->print_cr("In a coarsening: histo of occs:");
	  for (int k = 0; k < HistoSize; k++) {
	    gclog_or_tty->print_cr("  <= %4d: %5d.", histo_limits[k], histo[k]);
	  }
	  coarsenings++;
	  occ_sum += max_occ;
	  if ((coarsenings % 100) == 0) {
	    gclog_or_tty->print_cr("\ncoarsenings = %d; global summary:", coarsenings);
	    for (int k = 0; k < HistoSize; k++) {
	      gclog_or_tty->print_cr("  <= %4d: %5d.", histo_limits[k], global_histo[k]);
	    }
	    gclog_or_tty->print_cr("Avg occ of deleted region = %6.2f.",
	                  (float)occ_sum/(float)coarsenings);
	  }
	#endif
	
  {- -------------------------------------------
  (1) BitMap::at_put() を呼んで,
      _coarse_map 内の追い出し対象に該当する個所をマークしておく.
      (ついでに, _n_coarse_entries もインクリメントしておく)
  
      (ただし, 既に該当個所に印が付いていた場合には何もしない.
      <= これはマルチスレッドで race した場合?? #TODO)
      ---------------------------------------- -}

	  // Set the corresponding coarse bit.
	  int max_hrs_index = max->hr()->hrs_index();
	  if (!_coarse_map.at(max_hrs_index)) {
	    _coarse_map.at_put(max_hrs_index, true);
	    _n_coarse_entries++;
	#if 0
	    gclog_or_tty->print("Coarsened entry in region [" PTR_FORMAT "...] "
	               "for region [" PTR_FORMAT "...] (%d coarse entries).\n",
	               hr()->bottom(),
	               max->hr()->bottom(),
	               _n_coarse_entries);
	#endif
	  }
	
  {- -------------------------------------------
  (1) 追い出し対象をバケット内のリストから外す.
      ---------------------------------------- -}

	  // Unsplice.
	  *max_prev = max->next();

  {- -------------------------------------------
  (1) _n_coarsenings や _n_fine_entries の値を更新しておく.
      ---------------------------------------- -}

	  Atomic::inc(&_n_coarsenings);
	  _n_fine_entries--;

  {- -------------------------------------------
  (1) 選択した追い出し対象をリターン.
      ---------------------------------------- -}

	  return max;
	}
	
```


