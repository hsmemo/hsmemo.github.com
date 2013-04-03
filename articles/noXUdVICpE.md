---
layout: default
title: SpecializationStats クラス 
---
[Top](../index.html)

#### SpecializationStats クラス 



---
## <a name="nos79jVK1A" id="nos79jVK1A">SpecializationStats</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時以外には空のクラスとして定義される).

GC 中に以下のメソッドが何度実行されたかを計測し, 結果を表示するためのクラス
(特に, 特定の OopClosure に特化した _nv 版のメソッドとそうでないメソッドがそれぞれ何回呼ばれたか, を記録している).

  * Klass の oop_oop_iterate_*() メソッド
  * Closure の do_oop_*() メソッド, 
  * oopDesc の oop_iterate_*() メソッド


```
    ((cite: hotspot/src/share/vm/memory/specialized_oop_closures.hpp))
    class SpecializationStats {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
典型的な使用方法は以下のようになる模様.

  1. GC 処理の開始時に, SpecializationStats::clear() でカウンタを 0 にリセットする.
  2. GC 中に集計対象のメソッドが呼ばれる度に, そのメソッド内で対応する SpecializationStats::record_*() を呼び出してカウンタ値をインクリメントする.
  3. GC 処理の終了時に, SpecializationStats::print() で結果を表示させる.

### 内部構造(Internal structure)
内部には以下のようなメソッドを持つ.

ただし, 開発時用のクラスであるため #ifdef PRODUCT 時には全てのメソッドの中身が空になる
(PRODUCT_RETURN は #ifdef PRODUCT 時には '{}' に展開される. (See: PRODUCT_RETURN)).


```
    ((cite: hotspot/src/share/vm/memory/specialized_oop_closures.hpp))
      static void clear()  PRODUCT_RETURN;
    
      static inline void record_call()  PRODUCT_RETURN;
      static inline void record_iterate_call_v(Kind k)  PRODUCT_RETURN;
      static inline void record_iterate_call_nv(Kind k)  PRODUCT_RETURN;
      static inline void record_do_oop_call_v(Kind k)  PRODUCT_RETURN;
      static inline void record_do_oop_call_nv(Kind k)  PRODUCT_RETURN;
    
      static void print() PRODUCT_RETURN;
```

さらに, 開発時であっても ENABLE_SPECIALIZATION_STATS が 0 だとやっぱり空になる.


```
    ((cite: hotspot/src/share/vm/memory/specialized_oop_closures.hpp))
    #ifndef PRODUCT
    #if ENABLE_SPECIALIZATION_STATS
    ...
    #else   // !ENABLE_SPECIALIZATION_STATS
    
    inline void SpecializationStats::record_call() {}
    inline void SpecializationStats::record_iterate_call_v(Kind k) {}
    inline void SpecializationStats::record_iterate_call_nv(Kind k) {}
    inline void SpecializationStats::record_do_oop_call_v(Kind k) {}
    inline void SpecializationStats::record_do_oop_call_nv(Kind k) {}
    inline void SpecializationStats::clear() {}
    inline void SpecializationStats::print() {}
    
    #endif  // ENABLE_SPECIALIZATION_STATS
    #endif  // !PRODUCT
```

そして, 現状の ENABLE_SPECIALIZATION_STATS はデフォルトでは 0.

つまり, このクラスは基本的には使われない.


```
    ((cite: hotspot/src/share/vm/memory/specialized_oop_closures.hpp))
    // For keeping stats on effectiveness.
    #define ENABLE_SPECIALIZATION_STATS 0
```

### 備考(Notes)
なお, フィールドとしては以下のフィールド(のみ)が定義されている
(ただし, これらも #if ENABLE_SPECIALIZATION_STATS 時でないと定義さえされない).


```
    ((cite: hotspot/src/share/vm/memory/specialized_oop_closures.hpp))
    #if ENABLE_SPECIALIZATION_STATS
    private:
      static bool _init;
      static bool _wrapped;
      static jint _numCallsAll;
    
      static jint _numCallsTotal[NUM_Kinds];
      static jint _numCalls_nv[NUM_Kinds];
    
      static jint _numDoOopCallsTotal[NUM_Kinds];
      static jint _numDoOopCalls_nv[NUM_Kinds];
    public:
    #endif
```

以下のフィールドに各メソッド種別毎に呼ばれた回数を格納している 
(さらに _numCallsAll 以外については, 
呼び出し元を instanceKlass, instanceRefKlass, objArrayKlass の3つに分けて集計している).

  * _numCallsAll
    
    oopDesc::oop_iterate_*() 用
    
  * _numCallsTotal
    
    Klass::oop_oop_iterate_*() (_v/_nv 合計)

  * _numCalls_nv
    
    Klass::oop_oop_iterate_*() (_nv 版のみ)
    
  * _numDoOopCallsTotal

    Closure::do_oop_*() (_v/_nv 合計)
    
  * _numDoOopCalls_nv

    Closure::do_oop_*() (_nv 版のみ)




### 詳細(Details)
See: [here](../doxygen/classSpecializationStats.html) for details

---
