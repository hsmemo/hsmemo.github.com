---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp
### 説明(description)
(この関数は, 「領域中のどのくらいの割合を "dead wood" として扱えば良いか」を示す比率をリターンする.
 
 結果の比率は, 「領域中の現在の live オブジェクトの割合」に基づいた
 適当な正規分布を用いて計算している.
 (正確には, 正規分布で求めた値に少し調整を行い, 
  live オブジェクト割合が 100% の際に
  引数で指定された min_percent と等しくなるようにしている)
 
 なお, 参考までに 
 (正規分布の期待値が 0.5, 引数の min_percent が 1 と仮定した場合に)
 この関数が「正規分布の標準偏差」と「引数の density」に応じて
 どういう値を返すかを以下の表に示す.
 (正規分布なので, live オブジェクトの割合が 0.5 に近いほど dead wood 量は増える. 
  逆に, 0.5 から離れるほど dead wood 量は減る)


```
// Return a fraction indicating how much of the generation can be treated as
// "dead wood" (i.e., not reclaimed).  The function uses a normal distribution
// based on the density of live objects in the generation to determine a limit,
// which is then adjusted so the return value is min_percent when the density is
// 1.
//
// The following table shows some return values for a different values of the
// standard deviation (ParallelOldDeadWoodLimiterStdDev); the mean is 0.5 and
// min_percent is 1.
//
//                          fraction allowed as dead wood
//         -----------------------------------------------------------------
// density std_dev=70 std_dev=75 std_dev=80 std_dev=85 std_dev=90 std_dev=95
// ------- ---------- ---------- ---------- ---------- ---------- ----------
// 0.00000 0.01000000 0.01000000 0.01000000 0.01000000 0.01000000 0.01000000
// 0.05000 0.03193096 0.02836880 0.02550828 0.02319280 0.02130337 0.01974941
// 0.10000 0.05247504 0.04547452 0.03988045 0.03537016 0.03170171 0.02869272
// 0.15000 0.07135702 0.06111390 0.05296419 0.04641639 0.04110601 0.03676066
// 0.20000 0.08831616 0.07509618 0.06461766 0.05622444 0.04943437 0.04388975
// 0.25000 0.10311208 0.08724696 0.07471205 0.06469760 0.05661313 0.05002313
// 0.30000 0.11553050 0.09741183 0.08313394 0.07175114 0.06257797 0.05511132
// 0.35000 0.12538832 0.10545958 0.08978741 0.07731366 0.06727491 0.05911289
// 0.40000 0.13253818 0.11128511 0.09459590 0.08132834 0.07066107 0.06199500
// 0.45000 0.13687208 0.11481163 0.09750361 0.08375387 0.07270534 0.06373386
// 0.50000 0.13832410 0.11599237 0.09847664 0.08456518 0.07338887 0.06431510
// 0.55000 0.13687208 0.11481163 0.09750361 0.08375387 0.07270534 0.06373386
// 0.60000 0.13253818 0.11128511 0.09459590 0.08132834 0.07066107 0.06199500
// 0.65000 0.12538832 0.10545958 0.08978741 0.07731366 0.06727491 0.05911289
// 0.70000 0.11553050 0.09741183 0.08313394 0.07175114 0.06257797 0.05511132
// 0.75000 0.10311208 0.08724696 0.07471205 0.06469760 0.05661313 0.05002313
// 0.80000 0.08831616 0.07509618 0.06461766 0.05622444 0.04943437 0.04388975
// 0.85000 0.07135702 0.06111390 0.05296419 0.04641639 0.04110601 0.03676066
// 0.90000 0.05247504 0.04547452 0.03988045 0.03537016 0.03170171 0.02869272
// 0.95000 0.03193096 0.02836880 0.02550828 0.02319280 0.02130337 0.01974941
// 1.00000 0.01000000 0.01000000 0.01000000 0.01000000 0.01000000 0.01000000
```

### 名前(function name)
```
double PSParallelCompact::dead_wood_limiter(double density, size_t min_percent)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_dwl_initialized, "uninitialized");
	
  {- -------------------------------------------
  (1) 引数の density に対し, 
      領域中の (density*100)% が live オブジェクトである確率を計算する (以下の raw_limit).
      ---------------------------------------- -}

	  // The raw limit is the value of the normal distribution at x = density.
	  const double raw_limit = normal_distribution(density);
	
  {- -------------------------------------------
  (1) raw_limit の値を少し調整して, 
      density が 1 の場合に結果が min_percent になるようにし (以下の limit), 
      計算結果をリターンする.
      (なお, 値が 0 未満になってしまった場合は, 0.0 をリターンする)
      ---------------------------------------- -}

	  // Adjust the raw limit so it becomes the minimum when the density is 1.
	  //
	  // First subtract the adjustment value (which is simply the precomputed value
	  // normal_distribution(1.0)); this yields a value of 0 when the density is 1.
	  // Then add the minimum value, so the minimum is returned when the density is
	  // 1.  Finally, prevent negative values, which occur when the mean is not 0.5.
	  const double min = double(min_percent) / 100.0;
	  const double limit = raw_limit - _dwl_adjustment + min;
	  return MAX2(limit, 0.0);
	}
	
```


