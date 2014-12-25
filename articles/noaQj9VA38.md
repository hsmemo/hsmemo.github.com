---
layout: default
title: os クラス 
---
[Top](../index.html)

#### os クラス 



---
## <a name="nofRFxLZK1" id="nofRFxLZK1">os</a>

### 概要(Summary)
HotSpot 内から OS が提供する機能にアクセスするためのユーティリティ・クラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/share/vm/runtime/os.hpp))
    // os defines the interface to operating system; this includes traditional
    // OS services (time, I/O) as well as other functionality with system-
    // dependent code.
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/os.hpp))
    class os: AllStatic {
```

例えば次のような機能を提供している.

* 時間取得
* File/Socket I/O
* メモリ領域の確保/解放
* shared library のロード
* スレッドの作成/優先度制御
* signal handling, 
* 乱数生成
* プロセッサのコア数取得 & affinity 設定
* NUMA 設定
* etc

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
具体的な処理は os に依存するため, ほとんどのメソッドは os/ 及び os_cpu/ 以下で定義されている.




### 詳細(Details)
See: [here](../doxygen/classos.html) for details

---
