---
layout: default
title: JvmtiEnv クラス関連のクラス (JvmtiEnv, VM_JNIFunctionTableCopier)
---
[Top](../index.html)

#### JvmtiEnv クラス関連のクラス (JvmtiEnv, VM_JNIFunctionTableCopier)

これらは, JVMTI の機能を実装するために使われているクラス.
より具体的に言うと, "JVMTI environment" (JVMTI 環境) を実現するためのクラス.


### クラス一覧(class list)

  * [JvmtiEnv](#noFrbXDpxW)
  * [VM_JNIFunctionTableCopier](#noFQE5LFyR)


---
## <a name="noFrbXDpxW" id="noFrbXDpxW">JvmtiEnv</a>

### 概要(Summary)
JVMTI の "JVMTI environment" (JVMTI 環境) を実装するためのクラス.

1つの JvmtiEnv オブジェクトが 1つの JVMTI environment (= JVMTI エージェントが GetEnv() で取得する jvmtiEnv オブジェクト) に対応する (See: [here](no3718uqQ.html) for details).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiHpp.xsl))
    class JvmtiEnv : public JvmtiEnvBase {
```

(なお, JVMTI environment には JVMTI connection (JVMTI 接続) 毎の状態を管理する役割と
JVMTI 関数へのポインタ(environment pointer)を提供する役割があるが, どちらもこのクラスが担当している.
ただし, より正確に言うと JVMTI 接続毎の状態を管理する役割はスーパークラスである JvmtiEnvBase が担当している.
また, 一部の状態は JvmtiEnvThreadState クラスや JvmtiThreadState クラスが管理している. (See: [here](no3718uqQ.html) for details))

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
一般的には, GetEnv() で JVMTI environment を取得した各 JVMTI エージェント内に参照が保持されている.

またそれとは別に, 生成された全ての JvmtiEnv オブジェクトは
JvmtiEnvBase クラスの _head_environment フィールドに(線形リスト状になって)格納されている.

#### 生成箇所(where its instances are created)
JvmtiEnv::create_a_jvmti() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている (See: [here](no2935bUk.html) and [here](no30592Ee.html) for details).

```
jni_GetEnv()
-> JvmtiExport::get_jvmti_interface()
   -> JvmtiEnv::create_a_jvmti()
```

#### 削除箇所(where its instances are deleted)
JvmtiEnvBase::periodic_clean_up() 内で削除されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている. (See: JvmtiGCMarker)

```
JvmtiGCMarker::JvmtiGCMarker()
-> JvmtiEnvBase::check_for_periodic_clean_up()
   -> JvmtiEnvBase::periodic_clean_up()
```

### 内部構造(Internal structure)
内部には JVMTI の関数と同名のメソッドが定義されている (e.g. JvmtiEnv::Allocate(), JvmtiEnv::SetEventCallbacks(), etc).

JVMTI エージェントが環境ポインタ経由で JVMTI 関数を呼び出すと, これらのメソッドが呼び出される.

(正確には, まず jvmtiEnter.cpp で定義されている jvmti_*() 関数 
 (または jvmtiEnterTrace.cpp で定義されている jvmtiTrace_*() 関数) が呼び出され, 
 そこから JvmtiEnv のメソッドが呼び出される (See: [here](no3718uqQ.html) for details)).

### 備考(Notes)
このクラスの宣言は hotspot/src/share/vm/prims/jvmtiHpp.xsl から生成される.




### 詳細(Details)
See: [here](../doxygen/classJvmtiEnv.html) for details

---
## <a name="noFQE5LFyR" id="noFQE5LFyR">VM_JNIFunctionTableCopier</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラス(VM_Operationクラス).

JNI function table を指定されたものに書き換える.

(なお, 置き換えを非同期に行うと危険なので, safepoint 停止を利用して安全に置き換えられるように VM_Operation になっている).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiEnv.cpp))
    // VM operation class to copy jni function table at safepoint.
    // More than one java threads or jvmti agents may be reading/
    // modifying jni function tables. To reduce the risk of bad
    // interaction b/w these threads it is copied at safepoint.
    class VM_JNIFunctionTableCopier : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEnv::SetJNIFunctionTable() 内で(のみ)使用されている (See: [here](no2935l7p.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__JNIFunctionTableCopier.html) for details

---
