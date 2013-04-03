---
layout: default
title: JvmtiThreadState クラス関連のクラス (JvmtiEnvThreadStateIterator, JvmtiThreadState, RedefineVerifyMark)
---
[Top](../index.html)

#### JvmtiThreadState クラス関連のクラス (JvmtiEnvThreadStateIterator, JvmtiThreadState, RedefineVerifyMark)

これらは, JVMTI の機能を実装するために使われているクラス.
より具体的に言うと, "JVMTI environment" (JVMTI 環境) を実現するためのクラス.


### クラス一覧(class list)

  * [JvmtiThreadState](#nohjVW_Jos)
  * [JvmtiEnvThreadStateIterator](#noPdT6ve6X)
  * [RedefineVerifyMark](#noCCMDsdZX)


---
## <a name="nohjVW_Jos" id="nohjVW_Jos">JvmtiThreadState</a>

### 概要(Summary)
JVMTI の "JVMTI environment" (JVMTI 環境) を実装するためのクラス.

このクラスは JVMTI connection (JVMTI 接続) 毎の状態の管理を行う.
JVMTI 接続毎の状態の管理は, このクラスだけでなく, JvmtiEnvBase クラスや JvmtiEnvThreadState クラスと連携して行われている.
このクラスはその中で「スレッド毎に固有」な状態の管理を担当している
(See: [here](no3718uqQ.html) for details).

主に以下の情報を格納している.

  * 現在有効化されているイベント通知の種別(をスレッド単位で集計したもの)  (See: [here](no2935C7Z.html) for details)


```
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiThreadState
    //
    // The Jvmti state for each thread (across all JvmtiEnv):
    // 1. Local table of enabled events.
    class JvmtiThreadState : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 JavaThread オブジェクトの _jvmti_thread_state フィールド

  (ただし, オブジェクトの生成は実際に必要になるまで遅延されている)

* JvmtiThreadState クラスの _head フィールド

  生成された全ての JvmtiThreadState オブジェクトが (doubly-linked list 状になって)格納されている
  (各 JavaThread::_jvmti_thread_state フィールドから参照されている JvmtiThreadState を全てつないだもの).

#### 生成箇所(where its instances are created)
JvmtiThreadState::state_for_while_locked() 内で(のみ)生成されている
(= 初めて使用される時まで生成が遅延されている).

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
JvmtiEventControllerPrivate::recompute_enabled()
-> JvmtiThreadState::state_for_while_locked()

JvmtiEventControllerPrivate::thread_started()
-> JvmtiThreadState::state_for_while_locked()

JvmtiEventControllerPrivate::set_user_enabled()
-> JvmtiThreadState::state_for_while_locked()

JvmtiThreadState::state_for()
-> JvmtiThreadState::state_for_while_locked()
```

#### 削除箇所(where its instances are deleted)
JvmtiEventControllerPrivate::thread_ended() 内で削除されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
JavaThread::exit()
-> JvmtiExport::cleanup_thread()
   -> JvmtiEventController::thread_ended()
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiThreadState.html) for details

---
## <a name="noPdT6ve6X" id="noPdT6ve6X">JvmtiEnvThreadStateIterator</a>

### 概要(Summary)
JVMTI の機能を実装するために使われている補助クラス(StackObjクラス).

JvmtiThreadState 内に格納されている全ての JvmtiEnvThreadState オブジェクトをたどるためのイテレータクラス.

```
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiEnvThreadStateIterator
    //
    // The only safe means of iterating through the JvmtiEnvThreadStates
    // in a JvmtiThreadState.
    // Note that this iteratation includes invalid environments pending
    // deallocation -- in fact, some uses depend on this behavior.
    //
    class JvmtiEnvThreadStateIterator : public StackObj {
```

### 使われ方(Usage)
JVMTI 関連の様々な処理で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classJvmtiEnvThreadStateIterator.html) for details

---
## <a name="noCCMDsdZX" id="noCCMDsdZX">RedefineVerifyMark</a>

### 概要(Summary)
VM_RedefineClasses クラス内で使用される補助クラス(StackObjクラス).

Redefine 処理中でクラスファイルの verify (Verifier::verify()) を行う際に, 
その verify 処理の間だけ JvmtiThreadState オブジェクト内にクラス情報を入れておくために使用される.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.hpp))
    class RedefineVerifyMark : public StackObj {
```


```
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.hpp))
      // RedefineClasses support
      // The bug 6214132 caused the verification to fail.
      //
      // Below is the detailed description of the fix approach taken:
      // 1. What's done in RedefineClasses() before verification:
      //  a) A reference to the class being redefined (_the_class) and a
      //     reference to new version of the class (_scratch_class) are
      //     saved here for use during the bytecode verification phase of
      //     RedefineClasses. See RedefineVerifyMark for how these fields
      //     are managed.
      //   b) The _java_mirror field from _the_class is copied to the
      //     _java_mirror field in _scratch_class. This means that a jclass
      //     returned for _the_class or _scratch_class will refer to the
      //     same Java mirror. The verifier will see the "one true mirror"
      //     for the class being verified.
      // 2. What is done at verification:
      //   When the verifier makes calls into the VM to ask questions about
      //   the class being verified, it will pass the jclass to JVM_* functions.
      //   The jclass is always pointing to the mirror of _the_class.
      //   ~28 JVM_* functions called by the verifier for the information
      //   about CP entries and klass structure should check the jvmtiThreadState
      //   info about equivalent klass versions and use it to replace a klassOop
      //   of _the_class with a klassOop of _scratch_class. The function
      //   class_to_verify_considering_redefinition() must be called for it.
      //
      //   Note again, that this redirection happens only for the verifier thread.
      //   Other threads have very small overhead by checking the existence
      //   of the jvmtiThreadSate and the information about klasses equivalence.
      //   No JNI functions need to be changed, they don't reference the klass guts.
      //   The JavaThread pointer is already available in all JVM_* functions
      //   used by the verifier, so there is no extra performance issue with it.
```

### 使われ方(Usage)
VM_RedefineClasses::load_new_class_versions() 内で(のみ)使用されている (See: [here](no2935-Vj.html) for details).

### 内部構造(Internal structure)
コンストラクタで JvmtiThreadState::set_class_versions_map() を呼んでクラスの情報を一時的に書き込み, 
デストラクタで JvmtiThreadState::clear_class_versions_map() を呼んで元に戻している.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.hpp))
      RedefineVerifyMark(KlassHandle *the_class, KlassHandle *scratch_class,
                         JvmtiThreadState *state) : _state(state)
      {
        _state->set_class_versions_map(the_class, scratch_class);
        (*scratch_class)->set_java_mirror((*the_class)->java_mirror());
      }
    
      ~RedefineVerifyMark() {
        _state->clear_class_versions_map();
      }
```




### 詳細(Details)
See: [here](../doxygen/classRedefineVerifyMark.html) for details

---
