---
layout: default
title: SurvRateGroup クラス 
---
[Top](../index.html)

#### SurvRateGroup クラス 



---
## <a name="noRbdiVMCp" id="noRbdiVMCp">SurvRateGroup</a>

G1CollectorPolicy クラス内で使用される補助クラス.

毎回の GC で生き残ったオブジェクトの割合(生存率)を記録しておくためのクラス.
G1CollectorPolicy やその中で使用される CollectionSetChooser 内で使用されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/survRateGroup.hpp))
    class SurvRateGroup : public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. SurvRateGroup::start_adding_regions() で初期化する
2. SurvRateGroup::record_surviving_words() で情報を蓄積する
3. SurvRateGroup::stop_adding_regions() で _surv_rate_pred に結果を集計する(??)
4. SurvRateGroup::age_in_group() や SurvRateGroup::get_seq(), 
   SurvRateGroup::accum_surv_rate_pred(), SurvRateGroup::accum_surv_rate()
   等で計算結果を取得する(??)

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 G1CollectorPolicy オブジェクトの _short_lived_surv_rate_group フィールド
* 各 G1CollectorPolicy オブジェクトの _survivor_surv_rate_group フィールド

#### 生成箇所(where its instances are created)
G1CollectorPolicy::G1CollectorPolicy() 内で(のみ)生成されている.

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* G1CollectorPolicy * 	_g1p
* const char * 	_name
* size_t 	_stats_arrays_length
* double * 	_surv_rate
* double * 	_accum_surv_rate_pred
* double 	_last_pred
* double 	_accum_surv_rate
* TruncatedSeq ** 	_surv_rate_pred
* NumberSeq ** 	_summary_surv_rates
  
  デバッグ用(開発時用)のフィールド (備考参照).

* size_t 	_summary_surv_rates_len

  デバッグ用(開発時用)のフィールド (備考参照).

* size_t 	_summary_surv_rates_max_len

  デバッグ用(開発時用)のフィールド (備考参照).

* int 	_all_regions_allocated
* size_t 	_region_num
* size_t 	_setup_seq_num


### 備考(Notes)
develop オプションである G1YoungSurvRateNumRegionsSummary オプションが 1 以上の値に変更されている場合, 
以下のフィールドに NumberSeq オブジェクト (及びその長さ情報) が格納されるようになる
(G1YoungSurvRateNumRegionsSummary オプションが SurvRateGroup にコンストラクタ引数として渡されている. 
See: G1CollectorPolicy::G1CollectorPolicy()).

* SurvRateGroup::_summary_surv_rates
* SurvRateGroup::_summary_surv_rates_len
* SurvRateGroup::_summary_surv_rates_max_len

この NumerSeq に溜められた情報は SurvRateGroup::print_surv_rate_summary() でのみ使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
G1CollectedHeap::do_collection_pause_at_safepoint()
-&gt; G1CollectedHeap::print_tracing_info()          (&lt;= ExitAfterGCNum オプションが指定されている場合にのみ呼び出す)
   -&gt; G1CollectorPolicy::print_yg_surv_rate_info()
      -&gt; SurvRateGroup::print_surv_rate_summary() (&lt;= #ifndef PRODUCT 時にのみ呼び出す)
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classSurvRateGroup.html) for details

---
