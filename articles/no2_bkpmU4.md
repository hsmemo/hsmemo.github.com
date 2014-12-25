---
layout: default
title: AdaptiveSizePolicy クラス関連のクラス (AdaptiveSizePolicy, AdaptiveSizePolicyOutput)
---
[Top](../index.html)

#### AdaptiveSizePolicy クラス関連のクラス (AdaptiveSizePolicy, AdaptiveSizePolicyOutput)

これらは, Java ヒープ領域の大きさを動的に調整する際に使用されるクラス (See: [here](no28916PbD.html) for details).


### クラス一覧(class list)

  * [AdaptiveSizePolicy](#no43HROl7s)
  * [AdaptiveSizePolicyOutput](#no5o6XFrSM)


---
## <a name="no43HROl7s" id="no43HROl7s">AdaptiveSizePolicy</a>

### 概要(Summary)
Java ヒープの動的なサイズ変更処理で使用されるクラス.
新しいヒープサイズの計算を行う.

新しいヒープサイズを計算するクラスは使用する GC アルゴリズムによって異なるが, 
これらのクラスは GC アルゴリズムが Serial Old の場合に使用される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/adaptiveSizePolicy.hpp))
    // This class keeps statistical information and computes the
    // size of the heap.
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/adaptiveSizePolicy.hpp))
    class AdaptiveSizePolicy : public CHeapObj {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
GenCollectorPolicy::initialize_size_policy() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classAdaptiveSizePolicy.html) for details

---
## <a name="no5o6XFrSM" id="no5o6XFrSM">AdaptiveSizePolicyOutput</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する manageable オプションが指定されている場合にのみ使用される)
(See: PrintGCDetails).

AdaptiveSizePolicy に関する情報を出力するための一時オブジェクト(StackObjクラス).

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/adaptiveSizePolicy.hpp))
    // Class that can be used to print information about the
    // adaptive size policy at intervals specified by
    // AdaptiveSizePolicyOutputInterval.  Only print information
    // if an adaptive size policy is in use.
    class AdaptiveSizePolicyOutput : StackObj {
```

(なお, このクラスは (PrintGCDetails オプションに加えて) 
UseAdaptiveSizePolicy オプションや AdaptiveSizePolicyOutputInterval オプション等も設定されている場合にしか使用されない)
(See: AdaptiveSizePolicyOutput::print_test())

### 使われ方(Usage)
コード中で AdaptiveSizePolicyOutput 型の局所変数を宣言するだけ.

デストラクタ内で AdaptiveSizePolicy::print_adaptive_size_policy_on() を呼び出すことで出力を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/adaptiveSizePolicy.hpp))
      ~AdaptiveSizePolicyOutput() {
        if (_do_print) {
          assert(UseAdaptiveSizePolicy, "Should not be in use");
          _size_policy->print_adaptive_size_policy_on(gclog_or_tty);
        }
      }
```

なお, コンストラクタに引き渡すカウンタ値(後述)には total_collections() が用いられることが多い模様 (See: total_collections()).

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp))
    bool PSScavenge::invoke_no_policy() {
    ...
      AdaptiveSizePolicyOutput(size_policy, heap->total_collections());
```

### 内部構造(Internal structure)
常に出力するわけではなく,
コンストラクタで指定されたカウンタ値が AdaptiveSizePolicyOutputInterval の倍数の場合にのみ出力を行う
(この判定は AdaptiveSizePolicyOutput::print_test() で行われている).

通常の使われ方だと GC を AdaptiveSizePolicyOutputInterval 回実行したら1回出力されるようになる模様.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/adaptiveSizePolicy.hpp))
      AdaptiveSizePolicyOutput(uint count) {
        if (UseAdaptiveSizePolicy && (AdaptiveSizePolicyOutputInterval > 0)) {
          CollectedHeap* heap = Universe::heap();
          _size_policy = heap->size_policy();
          _do_print = print_test(count);
        } else {
          _size_policy = NULL;
          _do_print = false;
        }
      }
      AdaptiveSizePolicyOutput(AdaptiveSizePolicy* size_policy,
                               uint count) :
        _size_policy(size_policy) {
        if (UseAdaptiveSizePolicy && (AdaptiveSizePolicyOutputInterval > 0)) {
          _do_print = print_test(count);
        } else {
          _do_print = false;
        }
      }
```

#### 参考(for your information): AdaptiveSizePolicyOutput::print_test()
See: [here](no344Dib.html) for details



### 詳細(Details)
See: [here](../doxygen/classAdaptiveSizePolicyOutput.html) for details

---
