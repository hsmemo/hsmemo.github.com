---
layout: default
title: MonitorChunk クラス 
---
[Top](../index.html)

#### MonitorChunk クラス 



---
## <a name="noabde6XF4" id="noabde6XF4">MonitorChunk</a>

### 概要(Summary)
JIT Compiler 用のクラス (より正確に言うと, 脱最適化処理用のクラス).

脱最適化対象となったスタックフレーム内のモニター情報を一時的に待避しておくためのクラス (See: [here](no3420xYb.html) for details).

1つの MonitorChunk オブジェクトが 1つの (vframe 的な意味での) スタックフレーム中のモニター情報全体に対応する
(つまり Java レベルのメソッドと 1対1対応するような論理的なスタックフレーム.
 インライン展開された場合は実際のスタックフレームとは 1対1対応しない).

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 vframeArrayElement オブジェクトの _monitors フィールド
  
  その vframeArrayElement オブジェクトが示すスタックフレームに対応する MonitorChunk オブジェクト

* 各 JavaThread オブジェクトの _monitor_chunks フィールド

  (正確には, このフィールドは MonitorChunk の線形リストを格納するフィールド.
  MonitorChunk オブジェクトは _next フィールドで次の MonitorChunk オブジェクトを指せる構造になっている.
  現在脱最適化されている最中のスタックフレームは, 対応する MonitorChunk オブジェクトをこの線形リスト中に持つ)

#### 生成箇所(where its instances are created)
vframeArrayElement::fill_in() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* 脱最適化処理 (Deoptimization 処理)

  Deoptimization::fetch_unroll_info_helper()
  -> Deoptimization::create_vframeArray()
     -> vframeArray::allocate()
        -> vframeArray::fill_in()
           -> vframeArrayElement::fill_in()
```

なお, vframeArrayElement::fill_in() 内で JavaThread::_monitor_chunks への登録も行われている.

#### 使用箇所(where its instances are used)
JavaThread::_monitor_chunks フィールドは以下の箇所で(のみ)参照されている.

```
  JavaThread::is_lock_owned()
  -> JavaThread::monitor_chunks()

  JavaThread::oops_do()
  -> JavaThread::monitor_chunks()
```

#### 削除箇所(where its instances are deleted)
vframeArrayElement::free_monitors() 内で(のみ)削除されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* 脱最適化処理 (Deoptimization 処理)
  
  Deoptimization::unpack_frames()
  -> vframeArray::unpack_to_stack()
     -> vframeArray::deallocate_monitor_chunks()
        -> vframeArrayElement::free_monitors()
```

なお, vframeArrayElement::free_monitors() 内で JavaThread::_monitor_chunks からの登録解除も行われている.

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

(_monitors フィールドは BasicObjectLock の配列を指すポインタ. 
 配列長は _number_of_monitors フィールドに格納している.
 また, 配列用の領域はコンストラクタ中で確保している)


```cpp
    ((cite: hotspot/src/share/vm/runtime/monitorChunk.hpp))
      int              _number_of_monitors;
      BasicObjectLock* _monitors;
      BasicObjectLock* monitors() const { return _monitors; }
      MonitorChunk*    _next;
```




### 詳細(Details)
See: [here](../doxygen/classMonitorChunk.html) for details

---
