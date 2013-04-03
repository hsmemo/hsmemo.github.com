---
layout: default
title: BiasedLocking クラス関連のクラス (BiasedLockingCounters, BiasedLocking, 及びそれらの補助クラス(VM_EnableBiasedLocking, EnableBiasedLockingTask, VM_RevokeBias, VM_BulkRevokeBias))
---
[Top](../index.html)

#### BiasedLocking クラス関連のクラス (BiasedLockingCounters, BiasedLocking, 及びそれらの補助クラス(VM_EnableBiasedLocking, EnableBiasedLockingTask, VM_RevokeBias, VM_BulkRevokeBias))

これらは, 同期排他処理 (monitorenter/monitorexit 命令, synchronized 修飾子) のためのクラス.
より具体的に言うと, 同期排他処理を高速化する Biased Locking 機能のためのクラス
(See: [here](no2114NIs.html) and [here](no2480eGD.html) for details).


### クラス一覧(class list)

  * [BiasedLocking](#noWutJzBmw)
  * [VM_EnableBiasedLocking](#noFrwkYekn)
  * [EnableBiasedLockingTask](#noFMQrjepJ)
  * [VM_RevokeBias](#noEHegNqFW)
  * [VM_BulkRevokeBias](#noyVB_wAed)
  * [BiasedLockingCounters](#noQCVhlgKj)


---
## <a name="noWutJzBmw" id="noWutJzBmw">BiasedLocking</a>

### 概要(Summary)
Biased Locking 機能に関する処理を納めた名前空間(AllStatic クラス).
特に, Biased Locking 機能の初期化処理と slow-path 処理が納められている (See: [here](no2114NIs.html) and [here](no2480eGD.html) for details).

(なお, fast-path の処理はアセンブリで組まれたルーチンが使われるので, ここにはない)


```
    ((cite: hotspot/src/share/vm/runtime/biasedLocking.hpp))
    class BiasedLocking : AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classBiasedLocking.html) for details

---
## <a name="noFrwkYekn" id="noFrwkYekn">VM_EnableBiasedLocking</a>

### 概要(Summary)
BiasedLocking クラス内で使用される補助クラス(VM_Operationクラス).

Biased Locking 機能の初期化を行う.


```
    ((cite: hotspot/src/share/vm/runtime/biasedLocking.cpp))
    class VM_EnableBiasedLocking: public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* EnableBiasedLockingTask::task() (BiasedLockingStartupDelay オプションが 0 より大きい場合)
* BiasedLocking::init() (BiasedLockingStartupDelay オプションが 0 以下の場合)




### 詳細(Details)
See: [here](../doxygen/classVM__EnableBiasedLocking.html) for details

---
## <a name="noFMQrjepJ" id="noFMQrjepJ">EnableBiasedLockingTask</a>

### 概要(Summary)
BiasedLocking クラス内で使用される補助クラス.

Biased Locking 機能の初期化を行うための PeriodicTask クラス.
HotSpot の起動から BiasedLockingStartupDelay ミリ秒経過した時点で Biased Locking 機能の初期化を実行する.

(なお, このクラスは BiasedLockingStartupDelay オプションが 0 より大きい場合にのみ使用される.
 (See: BiasedLocking::init()))


```
    ((cite: hotspot/src/share/vm/runtime/biasedLocking.cpp))
    // One-shot PeriodicTask subclass for enabling biased locking
    class EnableBiasedLockingTask : public PeriodicTask {
```

### 使われ方(Usage)
BiasedLocking::init() 内で(のみ)生成されている.

### 内部構造(Internal structure)
実際の処理は VM_EnableBiasedLocking に丸投げしている.




### 詳細(Details)
See: [here](../doxygen/classEnableBiasedLockingTask.html) for details

---
## <a name="noEHegNqFW" id="noEHegNqFW">VM_RevokeBias</a>

### 概要(Summary)
BiasedLocking クラス内で使用される補助クラス(VM_Operationクラス).

他スレッドに Biased されているオブジェクトに対し, 
Safepoint 停止を使って安全に Bias を解除する(= revoke する).


```
    ((cite: hotspot/src/share/vm/runtime/biasedLocking.cpp))
    class VM_RevokeBias : public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* BiasedLocking::revoke()
* BiasedLocking::revoke_and_rebias()




### 詳細(Details)
See: [here](../doxygen/classVM__RevokeBias.html) for details

---
## <a name="noyVB_wAed" id="noyVB_wAed">VM_BulkRevokeBias</a>

### 概要(Summary)
BiasedLocking クラス内で使用される補助クラス(VM_Operationクラス).

Bulk Rebias 処理, 及び Bulk Revoke 処理を行う.


```
    ((cite: hotspot/src/share/vm/runtime/biasedLocking.cpp))
    class VM_BulkRevokeBias : public VM_RevokeBias {
```

### 使われ方(Usage)
BiasedLocking::revoke_and_rebias() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classVM__BulkRevokeBias.html) for details

---
## <a name="noQCVhlgKj" id="noQCVhlgKj">BiasedLockingCounters</a>

### 概要(Summary)
トラブルシューティング用/デバッグ用(開発時用)のクラス (関連する diagnostic オプションまたは notproduct オプションが指定されている場合にのみ使用される) 
(See: PrintBiasedLockingStatistics, PrintPreciseBiasedLockingStatistics, PrintLockStatistics).

Biased Locking 機能に関するプロファイル情報を溜めていくためのクラス.


```
    ((cite: hotspot/src/share/vm/runtime/biasedLocking.hpp))
    // Biased locking counters
    class BiasedLockingCounters VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* BiasedLocking クラスの _counters フィールド (static フィールド)
* 各 BiasedLockingNamedCounter オブジェクトの _counters フィールド

#### 生成箇所(where its instances are created)
* (BiasedLocking クラスの _counters フィールドは, ポインタ型ではなく実体なので,
   初期段階で自動的に生成される)

* (BiasedLockingNamedCounter クラスの _counters フィールドは, ポインタ型ではなく実体なので,
   BiasedLockingNamedCounter オブジェクトの生成時に一緒に生成される)

#### 情報の出力箇所(where the recorded information is output)
以下の箇所で(のみ)出力されている.

* BiasedLocking::print_counters()
* OptoRuntime::print_named_counters()




### 詳細(Details)
See: [here](../doxygen/classBiasedLockingCounters.html) for details

---
