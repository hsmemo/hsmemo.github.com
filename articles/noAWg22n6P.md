---
layout: default
title: InvocationCounter クラス 
---
[Top](../index.html)

#### InvocationCounter クラス 



---
## <a name="no7SL28FOG" id="no7SL28FOG">InvocationCounter</a>

### 概要(Summary)
JIT コンパイル処理のためのクラス. 
JIT コンパイルを開始するための閾値を管理する (See: [here](no7882MiN.html), [here](no2935G1h.html) and [here](no2935sgV.html) for details).

(JIT コンパイルは実行される回数が多いコードにのみ適用される.
InvocationCounter クラスはそのための閾値と実行回数を管理し, 実行回数が閾値に達したかどうかを判定する)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/invocationCounter.hpp))
    // InvocationCounters are used to trigger actions when a limit (threshold) is reached.
    // For different states, different limits and actions can be defined in the initialization
    // routine of InvocationCounters.
    //
    // Implementation notes: For space reasons, state & counter are both encoded in one word,
    // The state is encoded using some of the least significant bits, the counter is using the
    // more significant bits. The counter is incremented before a method is activated and an
    // action is triggered when when count() > limit().
    
    class InvocationCounter VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 methodOopDesc オブジェクトの _invocation_counter フィールド
* 各 methodOopDesc オブジェクトの _backedge_counter フィールド


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.hpp))
      InvocationCounter _invocation_counter;         // Incremented before each activation of the method - used to trigger frequency-based optimizations
      InvocationCounter _backedge_counter;           // Incremented before each backedge taken - used to trigger frequencey-based optimizations
```

* 各 methodDataOopDesc オブジェクトの _invocation_counter フィールド
* 各 methodDataOopDesc オブジェクトの _backedge_counter フィールド


```cpp
    ((cite: hotspot/src/share/vm/oops/methodDataOop.hpp))
      // How many invocations has this MDO seen?
      // These counters are used to determine the exact age of MDO.
      // We need those because in tiered a method can be concurrently
      // executed at different levels.
      InvocationCounter _invocation_counter;
      // Same for backedges.
      InvocationCounter _backedge_counter;
```

### 内部構造(Internal structure)
内部には実行回数をカウントする unsigned int フィールドを保持している.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/invocationCounter.hpp))
     private:                             // bit no: |31  3|  2  | 1 0 |
      unsigned int _counter;              // format: [count|carry|state]
```

また, 内部には 3種類の int 変数を保持している. これらに JIT コンパイル処理の閾値が格納されている.

* InterpreterInvocationLimit (メソッドコンパイル用)
* InterpreterBackwardBranchLimit (OSR 用)
* InterpreterProfileLimit (プロファイリング用)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/invocationCounter.hpp))
      static int InterpreterInvocationLimit;        // CompileThreshold scaled for interpreter use
      static int InterpreterBackwardBranchLimit;    // A separate threshold for on stack replacement
      static int InterpreterProfileLimit;           // Profiling threshold scaled for interpreter use
```

### 備考(Notes)
なお, 閾値の値は CompileThreshold オプション, InterpreterProfilePercentage オプション, 及び OnStackReplacePercentage オプションで変更できる.

それぞれ以下のように影響する.

* `InterpreterInvocationLimit = CompileThreshold << number_of_noncount_bits;`

* `InterpreterProfileLimit = ((CompileThreshold * InterpreterProfilePercentage) / 100)<< number_of_noncount_bits;`

* `InterpreterBackwardBranchLimit =`
  * [ProfileInterpreter 有効時]: 
    `(CompileThreshold * (OnStackReplacePercentage - InterpreterProfilePercentage)) / 100`
  * [ProfileInterpreter 無効時]: 
    `((CompileThreshold * OnStackReplacePercentage) / 100) << number_of_noncount_bits`


```cpp
    ((cite: hotspot/src/share/vm/interpreter/invocationCounter.cpp))
      InterpreterInvocationLimit = CompileThreshold << number_of_noncount_bits;
      InterpreterProfileLimit = ((CompileThreshold * InterpreterProfilePercentage) / 100)<< number_of_noncount_bits;
    
      // When methodData is collected, the backward branch limit is compared against a
      // methodData counter, rather than an InvocationCounter.  In the former case, we
      // don't need the shift by number_of_noncount_bits, but we do need to adjust
      // the factor by which we scale the threshold.
      if (ProfileInterpreter) {
        InterpreterBackwardBranchLimit = (CompileThreshold * (OnStackReplacePercentage - InterpreterProfilePercentage)) / 100;
      } else {
        InterpreterBackwardBranchLimit = ((CompileThreshold * OnStackReplacePercentage) / 100) << number_of_noncount_bits;
      }
```

### 備考(Notes)
なお, OSR に関連しそうな BackEdgeThreshold というオプションもあるが, どこからも使われていない...


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
      product_pd(intx, BackEdgeThreshold,                                       \
              "Interpreter Back edge threshold at which an OSR compilation is invoked")\
```




### 詳細(Details)
See: [here](../doxygen/classInvocationCounter.html) for details

---
