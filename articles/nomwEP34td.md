---
layout: default
title: vframe クラス関連のクラス (vframe, javaVFrame, interpretedVFrame, externalVFrame, entryVFrame, MonitorInfo, vframeStreamCommon, vframeStream)
---
[Top](../index.html)

#### vframe クラス関連のクラス (vframe, javaVFrame, interpretedVFrame, externalVFrame, entryVFrame, MonitorInfo, vframeStreamCommon, vframeStream)

これらは, HotSpot 内でのスレッド管理用のクラス.
より具体的に言うと, スレッドのスタックフレームに関する情報を取得するためのユーティリティ・クラス.

なお, これらのクラスは以下のような継承関係を持つ.


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    // The vframe inheritance hierarchy:
    // - vframe
    //   - javaVFrame
    //     - interpretedVFrame
    //     - compiledVFrame     ; (used for both compiled Java methods and native stubs)
    //   - externalVFrame
    //     - entryVFrame        ; special frame created when calling Java from C
```


### クラス一覧(class list)

  * [vframe](#noq_RugUu8)
  * [javaVFrame](#nooqyJLwQp)
  * [interpretedVFrame](#no2v0V_qFs)
  * [externalVFrame](#noCbnvVpCZ)
  * [entryVFrame](#noZB7fVaqh)
  * [MonitorInfo](#noZHL3Q4qQ)
  * [vframeStreamCommon](#no5otQlXjO)
  * [vframeStream](#nop7QqPojQ)


---
## <a name="noq_RugUu8" id="noq_RugUu8">vframe</a>

### 概要(Summary)
HotSpot 内でのスレッド管理用のユーティリティ・クラス.

スレッドのスタックフレームの情報を取得するための一時オブジェクト(ResourceObjクラス) (の基底クラス).
次のような情報を取得できる.

* そのスタックフレームの種別 (Interpreter 実行されているメソッドのフレーム, JIT 生成コードのフレーム, Java のメソッドに対応しないフレーム)
* そのスタックフレームの親に当たるフレーム
* etc

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

なお, 1つの vframe オブジェクトが 1つのスタックフレームに対応する.
ただし, インライン展開された場合は実際のスタックフレームが Java レベルのメソッドと1対1対応しないことがある
(実際のスタックフレーム 1 に対して Java レベルのメソッド n 個が対応).
vframe は Java レベルのメソッドと 1対1対応するような論理的なスタックフレームを表す
(つまり, 実際のスタックフレームとは1対1対応しないことがある).


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    // vframes are virtual stack frames representing source level activations.
    // A single frame may hold several source level activations in the case of
    // optimized code. The debugging stored with the optimized code enables
    // us to unfold a frame as a stack of vframes.
    // A cVFrame represents an activation of a non-java method.
```


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    class vframe: public ResourceObj {
```




### 詳細(Details)
See: [here](../doxygen/classvframe.html) for details

---
## <a name="nooqyJLwQp" id="nooqyJLwQp">javaVFrame</a>

### 概要(Summary)
vframe クラスのサブクラスの1つ.

このクラスは, Java のメソッドによるスタックフレームを表す 
(つまり, Interpreter 実行されているメソッドのフレーム, または JIT 生成コードのフレームを表す.
 そうでないフレームについては externalVFrame が担当する).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    class javaVFrame: public vframe {
```

### 内部構造(Internal structure)
(スーパークラスである vframe クラスのメソッドに加えて) 以下のような情報を取得するメソッドが追加されている.

* そのフレームに対応するメソッド (methodOop)
* そのフレーム内での現在の実行箇所(bci)
* そのフレーム内での現在の局所変数の値 (StackValueCollection)
* そのフレーム内での現在のオペランドスタックの値 (StackValueCollection)
* そのフレーム内で取得されているモニター (MonitorInfo)

(なお, 情報の取得だけではなくスタックフレーム内の値を変更するメソッド(javaVFrame::set_locals())も備えているが, 
 「これは JVMTI によるデバッグ機能のためのもので JIT 生成コードのフレームには効かないことに注意」とのこと)


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
      // JVM state
      virtual methodOop                    method()         const = 0;
      virtual int                          bci()            const = 0;
      virtual StackValueCollection*        locals()         const = 0;
      virtual StackValueCollection*        expressions()    const = 0;
      // the order returned by monitors() is from oldest -> youngest#4418568
      virtual GrowableArray<MonitorInfo*>* monitors()       const = 0;
    
      // Debugging support via JVMTI.
      // NOTE that this is not guaranteed to give correct results for compiled vframes.
      // Deoptimize first if necessary.
      virtual void set_locals(StackValueCollection* values) const = 0;
```




### 詳細(Details)
See: [here](../doxygen/classjavaVFrame.html) for details

---
## <a name="no2v0V_qFs" id="no2v0V_qFs">interpretedVFrame</a>

### 概要(Summary)
javaVFrame クラスの具象サブクラスの1つ.

このクラスは, Interpreter 実行されているメソッドのスタックフレーム用.


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    class interpretedVFrame: public javaVFrame {
```

### 使われ方(Usage)
vframe::new_vframe() というファクトリメソッドが用意されており, その中で(のみ)生成されている
(ただし, ResourceObjクラスなので一時的なオブジェクト).

そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
* 脱最適化処理 (Deoptimization 処理)
  
  Deoptimization::fetch_unroll_info_helper()
  -> vframe::new_vframe()

  Deoptimization::revoke_biases_of_monitors(JavaThread* thread, frame fr, RegisterMap* map)
  -> vframe::new_vframe()

  Deoptimization::revoke_biases_of_monitors(CodeBlob* cb)
  -> vframe::new_vframe()

  Deoptimization::uncommon_trap_inner()
  -> vframe::new_vframe()

  Deoptimization::unpack_frames()
  -> vframeArray::unpack_to_stack()
     -> vframeArrayElement::unpack_on_stack()
        -> vframe::new_vframe()

  Deoptimization::fetch_unroll_info_helper()
  -> vframe::sender()
     -> vframe::new_vframe()

  Deoptimization::revoke_biases_of_monitors()
  -> vframe::sender()
     -> vframe::new_vframe()

  Deoptimization::revoke_biases_of_monitors()
  -> vframe::sender()
     -> vframe::new_vframe()

* RFrame オブジェクトの生成処理

  InterpretedRFrame::InterpretedRFrame(frame fr, JavaThread* thread, RFrame*const callee)
  -> vframe::new_vframe()

  InterpretedRFrame::InterpretedRFrame(frame fr, JavaThread* thread, methodHandle m)
  -> vframe::new_vframe()

  CompiledRFrame::init()
  -> vframe::new_vframe()

* (#TODO)

  JavaThread::last_java_vframe()
  -> vframe::new_vframe()

  JavaThread::last_java_vframe()
  -> vframe::sender()
     -> vframe::new_vframe()

* (#TODO)

  vframe::top()
  -> vframe::sender()
     -> vframe::new_vframe()

  vframe::java_sender()
  -> vframe::sender()
     -> vframe::new_vframe()

* JVMTI による「root または指定したオブジェクトから辿れる範囲を再帰的に辿る」処理

  VM_HeapWalkOperation::collect_stack_roots()
  -> vframe::new_vframe()

  VM_HeapWalkOperation::collect_stack_roots()
  -> vframe::sender()
     -> vframe::new_vframe()

* JVMTI による「interp_only_mode」に遷移する処理
  
  VM_EnterInterpOnlyMode::doit()
  -> vframe::sender()
     -> vframe::new_vframe()

* 保守運用機能によるヒープダンプ処理

  VM_HeapDumper::do_thread()
  -> vframe::new_vframe()

  VM_HeapDumper::do_thread()
  -> vframe::sender()
     -> vframe::new_vframe()

  ThreadStackTrace::dump_stack_at_safepoint()
  -> vframe::sender()
     -> vframe::new_vframe()

* デバッグ用(開発時用)のチェック処理

  VMEntryWrapper::~VMEntryWrapper()
  -> InterfaceSupport::walk_stack()
     -> InterfaceSupport::walk_stack_from()
        -> vframe::sender()
           -> vframe::new_vframe()

* デバッグ用のトレース出力処理

  JavaThread::print_stack_on()
  -> vframe::sender()
     -> vframe::new_vframe()

  JavaThread::trace_stack_from()
  -> vframe::sender()
     -> vframe::new_vframe()

* デバッグ用のユーティリティ関数 (ps)

  * ps()
    -> vframe::new_vframe()
  * ps()
    -> vframe::sender()
       -> vframe::new_vframe()
```




### 詳細(Details)
See: [here](../doxygen/classinterpretedVFrame.html) for details

---
## <a name="noCbnvVpCZ" id="noCbnvVpCZ">externalVFrame</a>

### 概要(Summary)
vframe クラスの具象サブクラスの1つ.

このクラスは, 対応する Java のメソッドがないスタックフレームを表す 
(例えば, JavaCalls が生成する entry frame (dummy frame), 等.
 Java のメソッドによるスタックフレームについては javaVFrame が担当する)


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    class externalVFrame: public vframe {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
vframe::new_vframe() というファクトリメソッドが用意されており, その中で(のみ)生成されている
(ただし, ResourceObjクラスなので一時的なオブジェクト).

そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
* 脱最適化処理 (Deoptimization 処理)
  
  Deoptimization::fetch_unroll_info_helper()
  -> vframe::new_vframe()

  Deoptimization::revoke_biases_of_monitors(JavaThread* thread, frame fr, RegisterMap* map)
  -> vframe::new_vframe()

  Deoptimization::revoke_biases_of_monitors(CodeBlob* cb)
  -> vframe::new_vframe()

  Deoptimization::uncommon_trap_inner()
  -> vframe::new_vframe()

  Deoptimization::unpack_frames()
  -> vframeArray::unpack_to_stack()
     -> vframeArrayElement::unpack_on_stack()
        -> vframe::new_vframe()

  Deoptimization::fetch_unroll_info_helper()
  -> vframe::sender()
     -> vframe::new_vframe()

  Deoptimization::revoke_biases_of_monitors()
  -> vframe::sender()
     -> vframe::new_vframe()

  Deoptimization::revoke_biases_of_monitors()
  -> vframe::sender()
     -> vframe::new_vframe()

* RFrame オブジェクトの生成処理

  InterpretedRFrame::InterpretedRFrame(frame fr, JavaThread* thread, RFrame*const callee)
  -> vframe::new_vframe()

  InterpretedRFrame::InterpretedRFrame(frame fr, JavaThread* thread, methodHandle m)
  -> vframe::new_vframe()

  CompiledRFrame::init()
  -> vframe::new_vframe()

* (#TODO)

  JavaThread::last_java_vframe()
  -> vframe::new_vframe()

  JavaThread::last_java_vframe()
  -> vframe::sender()
     -> vframe::new_vframe()

* (#TODO)

  vframe::top()
  -> vframe::sender()
     -> vframe::new_vframe()

  vframe::java_sender()
  -> vframe::sender()
     -> vframe::new_vframe()

* JVMTI による「root または指定したオブジェクトから辿れる範囲を再帰的に辿る」処理

  VM_HeapWalkOperation::collect_stack_roots()
  -> vframe::new_vframe()

  VM_HeapWalkOperation::collect_stack_roots()
  -> vframe::sender()
     -> vframe::new_vframe()

* JVMTI による「interp_only_mode」に遷移する処理
  
  VM_EnterInterpOnlyMode::doit()
  -> vframe::sender()
     -> vframe::new_vframe()

* 保守運用機能によるヒープダンプ処理

  VM_HeapDumper::do_thread()
  -> vframe::new_vframe()

  VM_HeapDumper::do_thread()
  -> vframe::sender()
     -> vframe::new_vframe()

  ThreadStackTrace::dump_stack_at_safepoint()
  -> vframe::sender()
     -> vframe::new_vframe()

* デバッグ用(開発時用)のチェック処理

  VMEntryWrapper::~VMEntryWrapper()
  -> InterfaceSupport::walk_stack()
     -> InterfaceSupport::walk_stack_from()
        -> vframe::sender()
           -> vframe::new_vframe()

* デバッグ用のトレース出力処理

  JavaThread::print_stack_on()
  -> vframe::sender()
     -> vframe::new_vframe()

  JavaThread::trace_stack_from()
  -> vframe::sender()
     -> vframe::new_vframe()

* デバッグ用のユーティリティ関数 (ps)

  * ps()
    -> vframe::new_vframe()
  * ps()
    -> vframe::sender()
       -> vframe::new_vframe()
```




### 詳細(Details)
See: [here](../doxygen/classexternalVFrame.html) for details

---
## <a name="noZB7fVaqh" id="noZB7fVaqh">entryVFrame</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

(おそらく JavaCalls が生成する entry frame (dummy frame) 用のクラスだと思われる).


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    class entryVFrame: public externalVFrame {
```




### 詳細(Details)
See: [here](../doxygen/classentryVFrame.html) for details

---
## <a name="noZHL3Q4qQ" id="noZHL3Q4qQ">MonitorInfo</a>

### 概要(Summary)
HotSpot 内でのスレッド管理用のユーティリティ・クラス.

あるスタックフレーム内で取得されているモニターの情報を取得するための一時オブジェクト(ResourceObjクラス).
1つの MonitorInfo オブジェクトが 1つのモニターに対応する.

なお, javaVFrame (のサブクラス) 用の補助クラスという役割の他に, 
Platform MXBean の java.lang.management.MonitorInfo クラスを実装するという役割もある模様.


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    // A MonitorInfo is a ResourceObject that describes a the pair:
    // 1) the owner of the monitor
    // 2) the monitor lock
    class MonitorInfo : public ResourceObj {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ただし, ResourceObjクラスなので一時的なオブジェクト).

* interpretedVFrame::monitors()
* compiledVFrame::monitors()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

```
* Biased locking 関係の処理 (revoke/rebias 処理, GC 時の mark フィールドの待避処理)

  revoke_bias()
  -> get_or_compute_monitor_info()
     -> javaVFrame::monitors()

  bulk_revoke_or_rebias_at_safepoint()
  -> get_or_compute_monitor_info()
     -> javaVFrame::monitors()

  BiasedLocking::preserve_marks()
  -> javaVFrame::monitors()

* 脱最適化処理 (Deoptimization 処理)

  Deoptimization::revoke_biases_of_monitors(JavaThread* thread, frame fr, RegisterMap* map)
  -> collect_monitors()
     -> javaVFrame::monitors()

  Deoptimization::revoke_biases_of_monitors(CodeBlob* cb)
  -> collect_monitors()
     -> javaVFrame::monitors()

  Deoptimization::fetch_unroll_info_helper()
  -> javaVFrame::monitors()

  Deoptimization::fetch_unroll_info_helper()
  -> Deoptimization::create_vframeArray()
     -> vframeArray::allocate()
        -> vframeArray::fill_in()
           -> vframeArrayElement::fill_in()

* JVMTI によるモニター情報の取得処理

  JvmtiEnv::GetObjectMonitorUsage()
  -> JvmtiEnvBase::get_object_monitor_usage()
     -> JvmtiEnvBase::count_locked_objects()
        -> javaVFrame::monitors()
  -> VMThread::execute()
     -> (略) (See: [here](no2935qaz.html) for details)
        -> VM_GetObjectMonitorUsage::doit()
           -> JvmtiEnvBase::get_object_monitor_usage()
              -> (同上)

* JVMTI による取得済みのモニターに関する情報の取得処理

  JvmtiEnv::GetOwnedMonitorInfo()
  -> JvmtiEnvBase::get_owned_monitors()
     -> JvmtiEnvBase::get_locked_objects_in_frame()
        -> javaVFrame::monitors()
     
  JvmtiEnv::GetOwnedMonitorStackDepthInfo()
  -> JvmtiEnvBase::get_owned_monitors()
     -> javaVFrame::monitors()
  -> VMThread::execute()
     -> (略) (See: [here](no2935qaz.html) for details)
        -> VM_GetOwnedMonitorInfo::doit()
           -> JvmtiEnvBase::get_owned_monitors()
              -> (同上)

* JMM による java.lang.management.ThreadInfo オブジェクトの処理

  ThreadStackTrace::add_stack_frame()
  -> StackFrameInfo::StackFrameInfo()
     -> javaVFrame::locked_monitors()
        -> javaVFrame::monitors()

* デバッグ用のトレース出力処理

  JavaThread::print_stack_on()
  -> javaVFrame::print_lock_info_on()
     -> javaVFrame::monitors()

  javaVFrame::print()
  -> javaVFrame::monitors()
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(そして, メソッドはこれらのフィールドへの getter メソッド(アクセサメソッド)のみ).


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
      oop        _owner; // the object owning the monitor
      BasicLock* _lock;
      oop        _owner_klass; // klass if owner was scalar replaced
      bool       _eliminated;
      bool       _owner_is_scalar_replaced;
```




### 詳細(Details)
See: [here](../doxygen/classMonitorInfo.html) for details

---
## <a name="no5otQlXjO" id="no5otQlXjO">vframeStreamCommon</a>

### 概要(Summary)
スタック中のフレーム(vframe)をたどるためのイテレータクラス(StackObjクラス) (の基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    class vframeStreamCommon : StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classvframeStreamCommon.html) for details

---
## <a name="nop7QqPojQ" id="nop7QqPojQ">vframeStream</a>

### 概要(Summary)
vframeStreamCommon の具象サブクラスの 1つ.


```
    ((cite: hotspot/src/share/vm/runtime/vframe.hpp))
    class vframeStream : public vframeStreamCommon {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
実際の処理のほとんどはスーパークラスである vframeStreamCommon に丸投げしている
(このクラスで定義しているのはコンストラクタのみ).




### 詳細(Details)
See: [here](../doxygen/classvframeStream.html) for details

---
