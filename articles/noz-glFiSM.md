---
layout: default
title: ReferencePolicy クラス関連のクラス (ReferencePolicy, NeverClearPolicy, AlwaysClearPolicy, LRUCurrentHeapPolicy, LRUMaxHeapPolicy)
---
[Top](../index.html)

#### ReferencePolicy クラス関連のクラス (ReferencePolicy, NeverClearPolicy, AlwaysClearPolicy, LRUCurrentHeapPolicy, LRUMaxHeapPolicy)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, 参照オブジェクト (java.lang.ref オブジェクト) の GC 処理において
「いつ soft reference が GC されるべきか」という判断を行うクラス (See: [here](no289169tf.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/referencePolicy.hpp))
    // referencePolicy is used to determine when soft reference objects
    // should be cleared.
```

これらのクラスは以下のような継承関係を持つ
(ReferencePolicy クラスが基底クラスで, その下に異なる policy を実装した4つのサブクラスがある).

  * ReferencePolicy : 基底クラス
  	* NeverClearPolicy : (どんな場合でも) soft reference は決して消さない
  	* AlwaysClearPolicy : (どんな場合でも) soft reference は全て消去する
  	* LRUCurrentHeapPolicy : アクセス頻度が一定以下の soft reference を消去する
  	* LRUMaxHeapPolicy : アクセス頻度が一定以下の soft reference を消去する

なお, 判断は ReferencePolicy::should_clear_reference() が返す boolean 値で示される.


### 備考(Notes)
LRUCurrentHeapPolicy と LRUMaxHeapPolicy の場合は, 以下の値を使用して判断を行う
(timestamp フィールドの値がある閾値以前であればアクセス頻度が少ないと判断して消去する).

* java.lang.ref.SoftReference 内のフィールド
  * clock フィールド (なお, これは static フィールド) :
    最後に GC が実行された時間が格納されている. (この値は, ReferenceProcessor::init_statics() で初期化された後,
    ReferenceProcessor::process_discovered_references() 内で呼び出される
    ReferenceProcessor::update_soft_ref_master_clock() の中で更新されている.)
  * timestamp フィールド (なお, これはインスタンスフィールド) :
    最後にそのインスタンスがアクセスされた時間 (最後に get() が呼ばれた時間) が格納されている
* 直近の GC 終了時におけるヒープ空き容量
* SoftRefLRUPolicyMSPerMB オプションの値


```java
    ((cite: jdk/src/share/classes/java/lang/ref/SoftReference.java))
    public class SoftReference<T> extends Reference<T> {
    
        /**
         * Timestamp clock, updated by the garbage collector
         */
        static private long clock;
    
        /**
         * Timestamp updated by each invocation of the get method.  The VM may use
         * this field when selecting soft references to be cleared, but it is not
         * required to do so.
         */
        private long timestamp;
```



### クラス一覧(class list)

  * [ReferencePolicy](#nomZTQGt1X)
  * [NeverClearPolicy](#nowbPFxbm7)
  * [AlwaysClearPolicy](#noxGAXttc0)
  * [LRUCurrentHeapPolicy](#noUMYfbDOW)
  * [LRUMaxHeapPolicy](#noSLw758qo)


---
## <a name="nomZTQGt1X" id="nomZTQGt1X">ReferencePolicy</a>

### 概要(Summary)
soft reference の消去に関する判断を行うクラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/referencePolicy.hpp))
    class ReferencePolicy : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.



### 詳細(Details)
See: [here](../doxygen/classReferencePolicy.html) for details

---
## <a name="nowbPFxbm7" id="nowbPFxbm7">NeverClearPolicy</a>

### 概要(Summary)
ReferencePolicy クラスの具象サブクラスの1つ.

どんな場合でも soft reference は全て残す(消去しない).


```cpp
    ((cite: hotspot/src/share/vm/memory/referencePolicy.hpp))
    class NeverClearPolicy : public ReferencePolicy {
```

### 使われ方(Usage)
(このクラスは使用箇所が見当たらないが... #TODO)

### 内部構造(Internal structure)
その名の通り, should_clear_reference() メソッドは常に false を返す.


```cpp
    ((cite: hotspot/src/share/vm/memory/referencePolicy.hpp))
      bool should_clear_reference(oop p) { return false; }
```




### 詳細(Details)
See: [here](../doxygen/classNeverClearPolicy.html) for details

---
## <a name="noxGAXttc0" id="noxGAXttc0">AlwaysClearPolicy</a>

ReferencePolicy クラスの具象サブクラスの1つ.

どんな場合でも soft reference を全て消去する.


```cpp
    ((cite: hotspot/src/share/vm/memory/referencePolicy.hpp))
    class AlwaysClearPolicy : public ReferencePolicy {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ReferencePolicy クラスの _always_clear_soft_ref_policy フィールド (static フィールド) に(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      // . the "clear all" policy
      static ReferencePolicy*   _always_clear_soft_ref_policy;
```

#### 生成箇所(where its instances are created)
ReferenceProcessor::init_statics() 内で(のみ)生成されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
      _always_clear_soft_ref_policy = new AlwaysClearPolicy();
```

#### 使用箇所(where its instances are used)
ReferenceProcessor の処理中で使用されている (See: [here](no289169tf.html) for details).

なお AlwaysClearPolicy が使用されるかどうかは,
参照オブジェクト処理の開始前に呼び出される ReferenceProcessor::setup_policy() への引数で決定される.
ReferenceProcessor::setup_policy() に true が渡されると
_always_clear_soft_ref_policy の値 (= AlwaysClearPolicy オブジェクト) が使用される.
(逆に false が渡された場合は _default_soft_ref_policy の値 (= LRUCurrentHeapPolicy または LRUMaxHeapPolicy) が使われる)

#### 参考(for your information): ReferenceProcessor::setup_policy()
See: [here](no28916NLN.html) for details
### 内部構造(Internal structure)
その名の通り, should_clear_reference() メソッドは常に true を返す.


```cpp
    ((cite: hotspot/src/share/vm/memory/referencePolicy.hpp))
      bool should_clear_reference(oop p) { return true; }
```




### 詳細(Details)
See: [here](../doxygen/classAlwaysClearPolicy.html) for details

---
## <a name="noUMYfbDOW" id="noUMYfbDOW">LRUCurrentHeapPolicy</a>

ReferencePolicy クラスの具象サブクラスの1つ.

(メモリの消費速度と照らし合わせて) アクセスされる頻度が一定以下の soft reference を解放する.

なお, LRUMaxHeapPolicy との違いは, メモリの消費速度の計算時に
(ヒープの最大長ではなく)現在のヒープ長を基準とする点.


```cpp
    ((cite: hotspot/src/share/vm/memory/referencePolicy.hpp))
    class LRUCurrentHeapPolicy : public ReferencePolicy {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ReferencePolicy クラスの _default_soft_ref_policy フィールド (static フィールド) に(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      // . the default policy
      static ReferencePolicy*   _default_soft_ref_policy;
```

#### 生成箇所(where its instances are created)
ReferenceProcessor::init_statics() 内で(のみ)生成されている.

(なお LRUCurrentHeapPolicy が生成されるかどうかはビルド設定によって決まる.
COMPILER2_PRESENT の場合は LRUMaxHeapPolicy が生成され, そうでない場合は LRUCurrentHeapPolicy が生成される)


```cpp
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
      _default_soft_ref_policy      = new COMPILER2_PRESENT(LRUMaxHeapPolicy())
                                          NOT_COMPILER2(LRUCurrentHeapPolicy());
```

#### 使用箇所(where its instances are used)
ReferenceProcessor の処理中で使用されている (See: [here](no289169tf.html) for details).

なお LRUCurrentHeapPolicy が使用されるかどうかは,
参照オブジェクト処理の開始前に呼び出される ReferenceProcessor::setup_policy() への引数で決定される.
ReferenceProcessor::setup_policy() に false が渡されると
_default_soft_ref_policy の値 (= LRUCurrentHeapPolicy または LRUMaxHeapPolicy) が使用される.
(逆に true が渡された場合は _always_clear_soft_ref_policy の値 (= AlwaysClearPolicy オブジェクト) が使われる)

#### 参考(for your information): ReferenceProcessor::setup_policy()
See: [here](no28916NLN.html) for details

### 内部構造(Internal structure)
LRUCurrentHeapPolicy オブジェクトの生成時に, コンストラクタ内で基準となる時間(_max_interval)が計算されている.

そして実際の判断処理では,
判断対象の soft reference のアクセス間隔をその基準時間(_max_interval)と比較することで消去するかどうかを決めている.

#### 参考(for your information): LRUCurrentHeapPolicy::LRUCurrentHeapPolicy()
See: [here](no7882Jw1.html) for details
#### 参考(for your information): LRUCurrentHeapPolicy::setup()
See: [here](no28916tZQ.html) for details
#### 参考(for your information): LRUCurrentHeapPolicy::should_clear_reference()
See: [here](no28916Huc.html) for details



### 詳細(Details)
See: [here](../doxygen/classLRUCurrentHeapPolicy.html) for details

---
## <a name="noSLw758qo" id="noSLw758qo">LRUMaxHeapPolicy</a>

### 概要(Summary)
ReferencePolicy クラスの具象サブクラスの1つ.

(メモリの消費速度と照らし合わせて) アクセスされる頻度が一定以下の soft reference を解放する.

なお, LRUCurrentHeapPolicy との違いは, メモリの消費速度の計算時に
(現在のヒープ長ではなく)ヒープの最大長を基準とする点.


```cpp
    ((cite: hotspot/src/share/vm/memory/referencePolicy.hpp))
    class LRUMaxHeapPolicy : public ReferencePolicy {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ReferencePolicy クラスの _default_soft_ref_policy フィールド (static フィールド) に(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      // . the default policy
      static ReferencePolicy*   _default_soft_ref_policy;
```

#### 生成箇所(where its instances are created)
ReferenceProcessor::init_statics() 内で(のみ)生成されている.

(なお LRUMaxHeapPolicy が生成されるかどうかはビルド設定によって決まる.
COMPILER2_PRESENT の場合は LRUMaxHeapPolicy が生成され, そうでない場合は LRUCurrentHeapPolicy が生成される)


```cpp
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
      _default_soft_ref_policy      = new COMPILER2_PRESENT(LRUMaxHeapPolicy())
                                          NOT_COMPILER2(LRUCurrentHeapPolicy());
```

#### 使用箇所(where its instances are used)
ReferenceProcessor の処理中で使用されている (See: [here](no289169tf.html) for details).

なお LRUMaxHeapPolicy が使用されるかどうかは,
参照オブジェクト処理の開始前に呼び出される ReferenceProcessor::setup_policy() への引数で決定される.
ReferenceProcessor::setup_policy() に false が渡されると
_default_soft_ref_policy の値 (= LRUCurrentHeapPolicy または LRUMaxHeapPolicy) が使用される.
(逆に true が渡された場合は _always_clear_soft_ref_policy の値 (= AlwaysClearPolicy オブジェクト) が使われる)

#### 参考(for your information): ReferenceProcessor::setup_policy()
See: [here](no28916NLN.html) for details
### 内部構造(Internal structure)
LRUMaxHeapPolicy オブジェクトの生成時に, コンストラクタ内で基準となる時間(_max_interval)が計算されている.

そして実際の判断処理では,
判断対象の soft reference のアクセス間隔をその基準時間(_max_interval)と比較することで消去するかどうかを決めている.

#### 参考(for your information): LRUMaxHeapPolicy::LRUMaxHeapPolicy()
See: [here](no788275E.html) for details
#### 参考(for your information): LRUMaxHeapPolicy::setup()
See: [here](no289166qK.html) for details

#### 参考(for your information): LRUMaxHeapPolicy::should_clear_reference()
See: [here](no28916H1Q.html) for details



### 詳細(Details)
See: [here](../doxygen/classLRUMaxHeapPolicy.html) for details

---
