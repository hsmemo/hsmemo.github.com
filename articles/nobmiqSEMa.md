---
layout: default
title: GC に関係した諸々のユーティリティ・クラス (AdaptiveWeightedAverage, AdaptivePaddedAverage, AdaptivePaddedNoZeroDevAverage, LinearLeastSquareFit, GCPauseTimer)
---
[Top](../index.html)

#### GC に関係した諸々のユーティリティ・クラス (AdaptiveWeightedAverage, AdaptivePaddedAverage, AdaptivePaddedNoZeroDevAverage, LinearLeastSquareFit, GCPauseTimer)

これらは, Garbage Collection 処理用の諸々のユーティリティ・クラス (See: [here](no3718kvd.html) for details).

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
    // Catch-all file for utility classes
```


### クラス一覧(class list)

  * [AdaptiveWeightedAverage](#nocKfYjw1C)
  * [AdaptivePaddedAverage](#noLPAiMy2Y)
  * [AdaptivePaddedNoZeroDevAverage](#noumHBcot-)
  * [LinearLeastSquareFit](#nolX1FSumD)
  * [GCPauseTimer](#noPXm5Ihly)


---
## <a name="nocKfYjw1C" id="nocKfYjw1C">AdaptiveWeightedAverage</a>

### 概要(Summary)
Garbage Collection 処理用のユーティリティ・クラス.

収集した浮動小数点値の重み付き平均を計算する.

将来の値に対する予測値を出すために使われている
(例えば「これまでの GC で掛かった処理時間の平均を出したい」といった際に使用されている).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
    // A weighted average maintains a running, weighted average
    // of some float value (templates would be handy here if we
    // need different types).
    //
    // The average is adaptive in that we smooth it for the
    // initial samples; we don't use the weight until we have
    // enough samples for it to be meaningful.
    //
    // This serves as our best estimate of a future unknown.
    //
    class AdaptiveWeightedAverage : public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 新しい AdaptiveWeightedAverage オブジェクトを new で生成する
2. 新しい値が得られる度に, AdaptiveWeightedAverage::sample() で登録していく
3. それまでの重み付き平均が知りたくなったら, AdaptiveWeightedAverage::average() で取得する


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
      // Update data with a new sample.
      void sample(float new_sample);
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
      float    average() const       { return _average;       }
```

### 内部構造(Internal structure)
AdaptiveWeightedAverage が計算する重み付き平均は以下のようになる.

        x*sample  +  (1-x)*avg

ただしそれぞれの変数の意味は以下の通り.

  * x は, 重み(weight)  (<= 正確には, 重みは % で表されるのでそれを 100 で割ったもの)
  * sample は, 直近の値
  * avg は, それまでの値に基づく重み付き平均

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
      static inline float exp_avg(float avg, float sample,
                                   unsigned int weight) {
        assert(0 <= weight && weight <= 100, "weight must be a percent");
        return (100.0F - weight) * avg / 100.0F + weight * sample / 100.0F;
      }
```

### 備考(Notes)
サンプル数が少ない間は誤差も大きいので, 初めは指定された重み(以下のweight)は使わない.

(サンプル数が 100/weight に達するまでは, 代わりに 100/サンプル数 を重みとして用いる. つまり全てのサンプルを同じ重みで扱う)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.cpp))
    float AdaptiveWeightedAverage::compute_adaptive_average(float new_sample,
                                                            float average) {
      // We smooth the samples by not using weight() directly until we've
      // had enough data to make it meaningful. We'd like the first weight
      // used to be 1, the second to be 1/2, etc until we have 100/weight
      // samples.
      unsigned count_weight = 100/count();
      unsigned adaptive_weight = (MAX2(weight(), count_weight));
    
      float new_avg = exp_avg(average, new_sample, adaptive_weight);
    
      return new_avg;
    }
```




### 詳細(Details)
See: [here](../doxygen/classAdaptiveWeightedAverage.html) for details

---
## <a name="noLPAiMy2Y" id="noLPAiMy2Y">AdaptivePaddedAverage</a>

### 概要(Summary)
Garbage Collection 処理用のユーティリティ・クラス.

AdaptiveWeightedAverage のサブクラス.
重み付き平均(average)だけでなく, 
(サンプル点の平均からのずれも考慮して)水増しした平均値(padded average)も一緒に計算する.

将来の値に対する予測の上限値を出すために使われる.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
    // A weighted average that includes a deviation from the average,
    // some multiple of which is added to the average.
    //
    // This serves as our best estimate of an upper bound on a future
    // unknown.
    class AdaptivePaddedAverage : public AdaptiveWeightedAverage {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 新しい AdaptivePaddedAverage オブジェクトを new で生成する
2. 新しい値が得られる度に, AdaptivePaddedAverage::sample() で登録していく
3. それまでの重み付き平均が知りたくなったら, AdaptiveWeightedAverage::average() で取得する.
   水増しした平均値が知りたくなったら, AdaptivePaddedAverage::padded_average() で取得する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
      float padded_average() const         { return _padded_avg; }
```

### 内部構造(Internal structure)
AdaptivePaddedAverage が計算する水増し平均は以下のようになる

        average + padding*deviation

ただしそれぞれの変数の意味は以下の通り.

  * average は, AdaptiveWeightedAverage と同じ方法で計算される重み付き平均
  * padding は, コンストラクタ引数で指定される調整用のパラメータ
  * deviation は, 「サンプル点と重み付き平均の差(の絶対値)」を
    全てのサンプル点に対して (AdaptiveWeightedAverage の重み付き平均計算と同じやり方で) 平均化したもの.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.cpp))
    void AdaptivePaddedAverage::sample(float new_sample) {
      // Compute new adaptive weighted average based on new sample.
      AdaptiveWeightedAverage::sample(new_sample);
    
      // Now update the deviation and the padded average.
      float new_avg = average();
      float new_dev = compute_adaptive_average(fabsd(new_sample - new_avg),
                                               deviation());
      set_deviation(new_dev);
      set_padded_average(new_avg + padding() * new_dev);
      _last_sample = new_sample;
    }
```




### 詳細(Details)
See: [here](../doxygen/classAdaptivePaddedAverage.html) for details

---
## <a name="noumHBcot-" id="noumHBcot-">AdaptivePaddedNoZeroDevAverage</a>

### 概要(Summary)
Garbage Collection 処理用のユーティリティ・クラス.

AdaptivePaddedAverage のサブクラス.
AdaptivePaddedAverage とほぼ同じだが水増しの計算方法が異なり, 
サンプル点の値が 0 の場合には, その値は分散の計算には使用しない.

(コメントによると, 0 を計算に入れると水増しした平均値が急激に変化することがあるのでそれを防ぐため, とのこと)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
    // A weighted average that includes a deviation from the average,
    // some multiple of which is added to the average.
    //
    // This serves as our best estimate of an upper bound on a future
    // unknown.
    // A special sort of padded average:  it doesn't update deviations
    // if the sample is zero. The average is allowed to change. We're
    // preventing the zero samples from drastically changing our padded
    // average.
    class AdaptivePaddedNoZeroDevAverage : public AdaptivePaddedAverage {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 新しい AdaptivePaddedNoZeroDevAverage オブジェクトを new で生成する
2. 新しい値が得られる度に, AdaptivePaddedNoZeroDevAverage::sample() で登録していく
3. それまでの重み付き平均が知りたくなったら, AdaptiveWeightedAverage::average() で取得する.
   水増しした平均値が知りたくなったら, AdaptivePaddedAverage::padded_average() で取得する.

### 内部構造(Internal structure)
AdaptivePaddedAverage::sample() とほぼ同様.
ただし, 「サンプル点と重み付き平均の差(の絶対値)」は, サンプル点の値が 0 でないものについてのみ計算する.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.cpp))
    void AdaptivePaddedNoZeroDevAverage::sample(float new_sample) {
      // Compute our parent classes sample information
      AdaptiveWeightedAverage::sample(new_sample);
    
      float new_avg = average();
      if (new_sample != 0) {
        // We only create a new deviation if the sample is non-zero
        float new_dev = compute_adaptive_average(fabsd(new_sample - new_avg),
                                                 deviation());
    
        set_deviation(new_dev);
      }
      set_padded_average(new_avg + padding() * deviation());
      _last_sample = new_sample;
    }
```




### 詳細(Details)
See: [here](../doxygen/classAdaptivePaddedNoZeroDevAverage.html) for details

---
## <a name="nolX1FSumD" id="nolX1FSumD">LinearLeastSquareFit</a>

### 概要(Summary)
Garbage Collection 処理用のユーティリティ・クラス.

与えられたデータに対して, 最小二乗法で直線フィッティングを行うクラス

(つまり「切片(intercept)」と「傾き(slope)」の値を推定するクラス. `y = intercept + slope * x` という形の式の当てはめを行う)
        

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
    // Use a least squares fit to a set of data to generate a linear
    // equation.
    //              y = intercept + slope * x
    
    class LinearLeastSquareFit : public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 新しい LinearLeastSquareFit オブジェクトを new で生成する

2. 新しい値が得られる度に, LinearLeastSquareFit::update() で登録していく

3. 未知の値に対する予測値(新しい x に対する y の値)が知りたくなったら, LinearLeastSquareFit::y() を呼ぶ.
   直線の傾きが知りたくなったら, LinearLeastSquareFit::slope() または
   LinearLeastSquareFit::decrement_will_decrease() または LinearLeastSquareFit::increment_will_decrease() を呼ぶ.

   (decrement_will_decrease() と increment_will_decrease() は傾きの正負だけを教えてくれるメソッド.
   それぞれ傾きが正/負の時に true を, 逆の時に false を返す.)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
      void update(double x, double y);
      double y(double x);
      double slope() { return _slope; }
      // Methods to decide if a change in the dependent variable will
      // achive a desired goal.  Note that these methods are not
      // complementary and both are needed.
      bool decrement_will_decrease();
      bool increment_will_decrease();
```




### 詳細(Details)
See: [here](../doxygen/classLinearLeastSquareFit.html) for details

---
## <a name="noPXm5Ihly" id="noPXm5Ihly">GCPauseTimer</a>

### 概要(Summary)
Garbage Collection 処理用のユーティリティ・クラス.

elapsedTimer を作業途中のあるスコープの中でだけ止めておきたい, という場合に使われる補助クラス(StackObjクラス).

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
    class GCPauseTimer : StackObj {
```

### 内部構造(Internal structure)
指定された elapsedTimer オブジェクトに対して, 
コンストラクタで elapsedTimer::stop() を, デストラクタで elapsedTimer::start() を呼び出すだけ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcUtil.hpp))
      GCPauseTimer(elapsedTimer* timer) {
        _timer = timer;
        _timer->stop();
      }
      ~GCPauseTimer() {
        _timer->start();
      }
```




### 詳細(Details)
See: [here](../doxygen/classGCPauseTimer.html) for details

---
