---
layout: default
title: BCEscapeAnalyzer クラス (BCEscapeAnalyzer, 及びその補助クラス(BCEscapeAnalyzer::ArgumentMap, BCEscapeAnalyzer::StateInfo))
---
[Top](../index.html)

#### BCEscapeAnalyzer クラス (BCEscapeAnalyzer, 及びその補助クラス(BCEscapeAnalyzer::ArgumentMap, BCEscapeAnalyzer::StateInfo))



### クラス一覧(class list)

  * [BCEscapeAnalyzer](#nofJ4J0qtJ)
  * [BCEscapeAnalyzer::ArgumentMap](#norPJMZEfu)
  * [BCEscapeAnalyzer::StateInfo](#noGvFO19dS)


---
## <a name="nofJ4J0qtJ" id="nofJ4J0qtJ">BCEscapeAnalyzer</a>

### 概要(Summary)
C2 JIT Compiler 用の補助クラス (#ifdef COMPILER2 時にしか定義されない).

エスケープ解析(Escape Analysis)中に使用される一時オブジェクト(ResourceObjクラス).

呼び出し先が分かっている(= 動的ディスパッチが不要な)メソッド呼び出し箇所について, 
その引数や返値についてのエスケープ解析を行うクラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.hpp))
    // This class implements a fast, conservative analysis of effect of methods
    // on the escape state of their arguments.  The analysis is at the bytecode
    // level.
```


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.hpp))
    class BCEscapeAnalyzer : public ResourceObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. ciMethod::get_bcea() で, そのメソッドに対応する BCEscapeAnalyzer オブジェクトを取得する.

2. 以下のメソッドで, 引数や返値についてのエスケープ解析結果が得られる.

   * BCEscapeAnalyzer::is_arg_local()


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.hpp))
      // The given argument does not escape the callee.
      bool is_arg_local(int i) const {
```

   * BCEscapeAnalyzer::is_arg_stack()


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.hpp))
      // The given argument escapes the callee, but does not become globally
      // reachable.
      bool is_arg_stack(int i) const {
```

   * BCEscapeAnalyzer::is_arg_returned()


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.hpp))
      // The given argument does not escape globally, and may be returned.
      bool is_arg_returned(int i) const {
```

   * BCEscapeAnalyzer::is_return_local()


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.hpp))
      // True iff only input arguments are returned.
      bool is_return_local() const {
```

   * BCEscapeAnalyzer::is_return_allocated()


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.hpp))
      // True iff only newly allocated unescaped objects are returned.
      bool is_return_allocated() const {
```
   
3. また以下のメソッドで, 使用された依存性情報を表す Dependencies オブジェクトを取得できる.

   * BCEscapeAnalyzer::copy_dependencies()


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.hpp))
      // Copy dependencies from this analysis into "deps"
      void copy_dependencies(Dependencies *deps);
```

#### インスタンスの格納場所(where its instances are stored)
各 ciMethod オブジェクトの _bcea フィールドに(のみ)格納されている.

(ただし, BCEscapeAnalyzer オブジェクトの生成自体は実際に必要になるまで遅延されている)

(なお, ResourceObjクラスなので一時的なオブジェクト)

#### 生成箇所(where its instances are created)
ciMethod::get_bcea() 内で(のみ)生成されている (= 初めて使用される時まで生成が遅延されている).

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* ConnectionGraph::process_call_arguments()
* ConnectionGraph::process_call_result()




### 詳細(Details)
See: [here](../doxygen/classBCEscapeAnalyzer.html) for details

---
## <a name="norPJMZEfu" id="norPJMZEfu">BCEscapeAnalyzer::ArgumentMap</a>

### 概要(Summary)
BCEscapeAnalyzer クラス内で使用される補助クラス.

各局所変数やオペランドスタック中の各スロットについて, 
その位置にどの引数の値が格納されうるかを記録しておくためのクラス.

1つの BCEscapeAnalyzer::ArgumentMap オブジェクトが 
1つの局所変数または 1つのオペランドスタック中のスロットに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.cpp))
    // Maintain a map of which aguments a local variable or
    // stack slot may contain.  In addition to tracking
    // arguments, it tracks two special values, "allocated"
    // which represents any object allocated in the current
    // method, and "unknown" which is any other object.
    // Up to 30 arguments are handled, with the last one
    // representing summary information for any extra arguments
    class BCEscapeAnalyzer::ArgumentMap {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 StateInfo オブジェクトの _vars フィールド
  
  (正確には, このフィールドは BCEscapeAnalyzer::ArgumentMap の配列を格納するフィールド.
  この中に, 局所変数に対応する全ての BCEscapeAnalyzer::ArgumentMap オブジェクトが格納されている)
   
* 各 StateInfo オブジェクトの _stack フィールド
  
  (正確には, このフィールドは BCEscapeAnalyzer::ArgumentMap の配列を格納するフィールド.
  この中に, オペランドスタック中のスロットに対応する全ての BCEscapeAnalyzer::ArgumentMap オブジェクトが格納されている)

* 各 StateInfo オブジェクトの empty_map フィールド

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* (StateInfo クラスの empty_map フィールドは, ポインタ型ではなく実体なので,
  StateInfo オブジェクトの生成時に一緒に生成される)

* BCEscapeAnalyzer::iterate_blocks()
  
  Arena::Amalloc() で BCEscapeAnalyzer::StateInfo の配列が確保される
  (ただし, Arena::Amalloc() なので一時的なオブジェクト)

* BCEscapeAnalyzer::iterate_one_block() (局所変数として生成)
* BCEscapeAnalyzer::compute_escape_for_intrinsic() (局所変数として生成)
* BCEscapeAnalyzer::merge_block_states() (局所変数として生成)
* BCEscapeAnalyzer::iterate_blocks() (局所変数として生成)
* BCEscapeAnalyzer::clear_escape_info() (局所変数として生成)

### 内部構造(Internal structure)
なお, operator= がオーバーライドされている
(このため StateInfo::raw_push() した結果はコピーされることに注意).


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.cpp))
      void operator=(const ArgumentMap &am) { _bits = am._bits; }
```




### 詳細(Details)
See: [here](../doxygen/classBCEscapeAnalyzer_1_1ArgumentMap.html) for details

---
## <a name="noGvFO19dS" id="noGvFO19dS">BCEscapeAnalyzer::StateInfo</a>

### 概要(Summary)
BCEscapeAnalyzer クラス内で使用される補助クラス.

各基本ブロックの先頭における局所変数／オペランドスタックの状態を示す.


```cpp
    ((cite: hotspot/src/share/vm/ci/bcEscapeAnalyzer.cpp))
    class BCEscapeAnalyzer::StateInfo {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
BCEscapeAnalyzer::iterate_blocks() 内で(のみ)生成されている.

(なお, 局所変数として生成されているほか, 
Arena::Amalloc() で BCEscapeAnalyzer::StateInfo の配列も確保されている.
ただし, Arena::Amalloc() なので一時的なオブジェクト)




### 詳細(Details)
See: [here](../doxygen/classBCEscapeAnalyzer_1_1StateInfo.html) for details

---
