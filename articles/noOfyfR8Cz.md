---
layout: default
title: NMethodSweeper クラス (NMethodSweeper, 及びその補助クラス(SweeperRecord, MarkActivationClosure, NMethodMarker))
---
[Top](../index.html)

#### NMethodSweeper クラス (NMethodSweeper, 及びその補助クラス(SweeperRecord, MarkActivationClosure, NMethodMarker))

これらは, JIT Compiler 用のクラス.
より具体的に言うと, 不要になった JIT 生成コード(nmethod)を回収するためのクラス
(See: [here](no1904IaU.html) for details).


### クラス一覧(class list)

  * [NMethodSweeper](#nob4T0Ogpi)
  * [SweeperRecord](#noz8QdJ_bL)
  * [MarkActivationClosure](#no2b3j_zZ3)
  * [NMethodMarker](#novHBxue_1)


---
## <a name="nob4T0Ogpi" id="nob4T0Ogpi">NMethodSweeper</a>

### 概要(Summary)
不要になった JIT 生成コード(nmethod)を回収するためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス))
(See: [here](no1904IaU.html) for details).

回収対象は, アンロードされたメソッドや "not entrant" になった nmethod, 
zombie 化された nmethod, 及びそれらを指している Inline cache, 等.


```
    ((cite: hotspot/src/share/vm/runtime/sweeper.hpp))
    // An NmethodSweeper is an incremental cleaner for:
    //    - cleanup inline caches
    //    - reclamation of unreferences zombie nmethods
    //
    
    class NMethodSweeper : public AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* 回収処理の開始 (毎回の Safepoint 時の実施)

  SafepointSynchronize::do_cleanup_tasks()
  -> NMethodSweeper::scan_stacks()

* CompilerBroker によるインクリメンタルな回収処理

  CompileQueue::get()
  -> NMethodSweeper::possibly_sweep()

* nmethod に変化があった場合の処理

  nmethod::make_unloaded()
  -> NMethodSweeper::notify()

  nmethod::make_not_entrant_or_zombie()
  -> NMethodSweeper::notify()

* nmethod の zombie 化のための処理

  nmethod::mark_as_seen_on_stack()
  -> NMethodSweeper::traversal_count()

  nmethod::can_not_entrant_be_converted()
  -> NMethodSweeper::traversal_count()

* CodeCache が一杯になった場合の処理

  CompileBroker::compiler_thread_loop()
  -> CompileBroker::handle_full_code_cache()
     -> NMethodSweeper::handle_full_code_cache()
        -> VMThread::execute()
           -> (略) (See: [here](no2935qaz.html) for details)
              -> VM_HandleFullCodeCache::doit()
                 -> NMethodSweeper::speculative_disconnect_nmethods()
  -> NMethodSweeper::handle_full_code_cache()
     -> (同上)

  ciEnv::register_method()
  -> CompileBroker::handle_full_code_cache()
     -> (同上)

  AdapterHandlerLibrary::get_adapter()
  -> CompileBroker::handle_full_code_cache()
     -> (同上)

  AdapterHandlerLibrary::create_native_wrapper()
  -> CompileBroker::handle_full_code_cache()
     -> (同上)
```
  



### 詳細(Details)
See: [here](../doxygen/classNMethodSweeper.html) for details

---
## <a name="noz8QdJ_bL" id="noz8QdJ_bL">SweeperRecord</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

NMethodSweeper クラス内で使用される補助クラス.
NMethodSweeper クラスに関するログ情報の記録／出力を行う.


```
    ((cite: hotspot/src/share/vm/runtime/sweeper.cpp))
    #ifdef ASSERT
```


```
    ((cite: hotspot/src/share/vm/runtime/sweeper.cpp))
    // Sweeper logging code
    class SweeperRecord {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
_records という大域変数に(のみ)格納されている.

(正確には, このフィールドは SweeperRecord の配列を格納するフィールド.
この中に, 使用される全ての SweeperRecord オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
NMethodSweeper::possibly_sweep() 内で(のみ)生成されている.

なお, このクラスは (デバッグ時であることに加えて) LogSweeper オプションが指定されている場合にしか生成されない.

#### 情報の記録箇所(where information is recorded)
現在は, NMethodSweeper::record_sweep() 内で(のみ)記録されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
SWEEP() マクロ
-> NMethodSweeper::record_sweep()
```


```
    ((cite: hotspot/src/share/vm/runtime/sweeper.cpp))
    #define SWEEP(nm) record_sweep(nm, __LINE__)
```

#### 情報の出力箇所(where the recorded information is output)
以下の箇所で(のみ)出力されている.

* NMethodSweeper::report_events(int id, address entry)
* NMethodSweeper::report_events()




### 詳細(Details)
See: [here](../doxygen/classSweeperRecord.html) for details

---
## <a name="no2b3j_zZ3" id="no2b3j_zZ3">MarkActivationClosure</a>

### 概要(Summary)
NMethodSweeper クラス内で使用される補助クラス.

スタックフレームのスキャン処理中に
「もう "not entrant" になっているメソッドのフレーム」
を見つけたら印を入れる Closure クラス


```
    ((cite: hotspot/src/share/vm/runtime/sweeper.cpp))
    class MarkActivationClosure: public CodeBlobClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
mark_activation_closure という大域変数に(のみ)格納されている.

#### 生成箇所(where its instances are created)
(mark_activation_closure 大域変数は, ポインタ型ではなく実体なので,
 初期段階で自動的に生成される)

#### 使用箇所(where its instances are used)
NMethodSweeper::scan_stacks() 内で(のみ)使用されている.

### 内部構造(Internal structure)
行う処理は「nmethod でかつ not entrant になっているメソッドのフレーム」を見つけたら印を入れるだけ.

#### 参考(for your information): MarkActivationClosure::do_code_blob()
See: [here](no171194wX.html) for details



### 詳細(Details)
See: [here](../doxygen/classMarkActivationClosure.html) for details

---
## <a name="novHBxue_1" id="novHBxue_1">NMethodMarker</a>

### 概要(Summary)
NMethodSweeper クラス内で使用される補助クラス.

ソースコード中のあるスコープの間だけ, 
処理中の nmethod がアンロードされないようにしておくためのクラス(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/runtime/sweeper.cpp))
    class NMethodMarker: public StackObj {
```

### 使われ方(Usage)
NMethodSweeper::process_nmethod() 内で(のみ)使用されている.

### 内部構造(Internal structure)
コンストラクタで CompilerThread::_scanned_nmethod フィールドに処理対象の nmethod をセットし, 
デストラクタで NULL に戻しているだけ.


```
    ((cite: hotspot/src/share/vm/runtime/sweeper.cpp))
      NMethodMarker(nmethod* nm) {
        _thread = CompilerThread::current();
        _thread->set_scanned_nmethod(nm);
      }
      ~NMethodMarker() {
        _thread->set_scanned_nmethod(NULL);
      }
```




### 詳細(Details)
See: [here](../doxygen/classNMethodMarker.html) for details

---
