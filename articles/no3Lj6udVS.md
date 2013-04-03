---
layout: default
title: compiledVFrame クラス関連のクラス (compiledVFrame, jvmtiDeferredLocalVariableSet, jvmtiDeferredLocalVariable)
---
[Top](../index.html)

#### compiledVFrame クラス関連のクラス (compiledVFrame, jvmtiDeferredLocalVariableSet, jvmtiDeferredLocalVariable)

これらは, HotSpot 内でのスレッド管理用のクラス.
より具体的に言うと, スレッドのスタックフレームに関する情報を取得するためのユーティリティ・クラス
(なお JVMTI による局所変数の変更処理を補佐する役割もある).


### クラス一覧(class list)

  * [compiledVFrame](#noObMh0HEN)
  * [jvmtiDeferredLocalVariableSet](#noyCPZpcmB)
  * [jvmtiDeferredLocalVariable](#noSgoAvLR-)


---
## <a name="noObMh0HEN" id="noObMh0HEN">compiledVFrame</a>

### 概要(Summary)
javaVFrame クラスの具象サブクラスの1つ.

このクラスは, JIT 生成コードのスタックフレーム用.


```
    ((cite: hotspot/src/share/vm/runtime/vframe_hp.hpp))
    class compiledVFrame: public javaVFrame {
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
See: [here](../doxygen/classcompiledVFrame.html) for details

---
## <a name="noyCPZpcmB" id="noyCPZpcmB">jvmtiDeferredLocalVariableSet</a>

### 概要(Summary)
保守運用機能のためのクラス (JVMTI 機能用のクラス).
より具体的に言うと, 局所変数の変更用の JVMTI 関数 (SetLocal*()) 用の補助クラス (See: [here](no2935GIU.html) for details).

変更対象が JIT 生成されたコードのフレーム内にある場合, 
そのままでは変更が難しいので脱最適化してから変更することになる.
jvmtiDeferredLocalVariableSet クラスは, 脱最適化が完了するまで変更予定を記録しておくためのクラス.

1つの jvmtiDeferredLocalVariableSet オブジェクトが 1つの (vframe 的な意味での) スタックフレームに対応する
(つまり Java レベルのメソッドと 1対1対応するような論理的なスタックフレームに対応. 
 インライン展開された場合は実際のスタックフレームとは 1対1対応しない).


```
    ((cite: hotspot/src/share/vm/runtime/vframe_hp.hpp))
    // In order to implement set_locals for compiled vframes we must
    // store updated locals in a data structure that contains enough
    // information to recognize equality with a vframe and to store
    // any updated locals.
```


```
    ((cite: hotspot/src/share/vm/runtime/vframe_hp.hpp))
    class jvmtiDeferredLocalVariableSet : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JavaThread オブジェクトの _deferred_locals_updates フィールドに(のみ)格納されている.
 
(正確には, このフィールドは jvmtiDeferredLocalVariableSet の GrowableArray を格納するフィールド.
この中に, その JavaThread 用の全ての jvmtiDeferredLocalVariableSet オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
compiledVFrame::update_local() 内で(のみ)生成されている.
GrowableArray 用のメモリ領域も compiledVFrame::update_local() 内で(のみ)確保されている. 

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
VM_GetOrSetLocal::doit()
-> compiledVFrame::update_local()
```

#### 使用箇所(where its instances are used)
compiledVFrame::locals() 内で(のみ)参照されている.

(これが vframeArrayElement::fill_in() から呼び出されるので, 
脱最適化が終わった後には更新後の値がスタックフレームには入っていることになる)

### 内部構造(Internal structure)
実際の更新情報は jvmtiDeferredLocalVariable 内に格納されている.




### 詳細(Details)
See: [here](../doxygen/classjvmtiDeferredLocalVariableSet.html) for details

---
## <a name="noSgoAvLR-" id="noSgoAvLR-">jvmtiDeferredLocalVariable</a>

### 概要(Summary)
jvmtiDeferredLocalVariableSet クラス内で使用される補助クラス.

実際に JVMTI による変更予定を記録しているクラス.
1つの jvmtiDeferredLocalVariableSet オブジェクトが 1つの局所変数に対応する.


```
    ((cite: hotspot/src/share/vm/runtime/vframe_hp.hpp))
    class jvmtiDeferredLocalVariable : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 jvmtiDeferredLocalVariableSet オブジェクトの _locals フィールドに(のみ)格納されている.
 
(正確には, このフィールドは jvmtiDeferredLocalVariable の GrowableArray を格納するフィールド.
この中に, その JavaThread 用の全ての jvmtiDeferredLocalVariableSet オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
compiledVFrame::update_local() 内で(のみ)生成されている.

(より正確には, compiledVFrame::update_local() 内と, 
そこから呼び出される jvmtiDeferredLocalVariableSet::set_local_at() 内で(のみ)生成されている)

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
VM_GetOrSetLocal::doit()
-> compiledVFrame::update_local()
```

なお, jvmtiDeferredLocalVariableSet::_locals フィールドの
GrowableArray 自体は jvmtiDeferredLocalVariableSet::jvmtiDeferredLocalVariableSet() 内で(のみ)確保されている. 




### 詳細(Details)
See: [here](../doxygen/classjvmtiDeferredLocalVariable.html) for details

---
