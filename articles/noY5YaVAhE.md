---
layout: default
title: JvmtiRawMonitor クラス関連のクラス (JvmtiRawMonitor, JvmtiPendingMonitors)
---
[Top](../index.html)

#### JvmtiRawMonitor クラス関連のクラス (JvmtiRawMonitor, JvmtiPendingMonitors)

これらは, JVMTI の関数を実装するために使われているクラス.
より具体的に言うと, RawMonitor 関係の関数
(CreateRawMonitor() 関数, DestroyRawMonitor() 関数,
RawMonitorEnter() 関数, RawMonitorExit() 関数,
RawMonitorWait() 関数, RawMonitorNotify() 関数, RawMonitorNotifyAll() 関数)
を実装するためのクラス (See: [here](no2935uzr.html) for details).


### クラス一覧(class list)

  * [JvmtiRawMonitor](#nog41pSUSE)
  * [JvmtiPendingMonitors](#noy77BYxt0)


---
## <a name="nog41pSUSE" id="nog41pSUSE">JvmtiRawMonitor</a>

### 概要(Summary)
JVMTI の関数 (より具体的に言うと, RawMonitor 関係の関数) を実装するためのクラス (See: [here](no2935uzr.html) for details).

1つの JvmtiRawMonitor オブジェクトが 1つの RawMonitor オブジェクト (= 1つの jrawMonitorID) に対応する.

(なお, 実装は Objectmonitor のサブクラスとして定義されている.
 RawMonitor が (再帰的な確保が可能等) ObjectMonitor によく似た性質を持っているため?? #TODO).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiRawMonitor.hpp))
    // class JvmtiRawMonitor
    //
    // Used by JVMTI methods: All RawMonitor methods (CreateRawMonitor, EnterRawMonitor, etc.)
    //
    // Wrapper for ObjectMonitor class that saves the Monitor's name
    //
    
    class JvmtiRawMonitor : public ObjectMonitor  {
```

### 使われ方(Usage)
JvmtiEnv の RawMonitor 関係のメソッド内で(のみ)使用されている
(JvmtiEnv::CreateRawMonitor(), JvmtiEnv::DestroyRawMonitor(),
JvmtiEnv::RawMonitorEnter(), JvmtiEnv::RawMonitorExit(),
JvmtiEnv::RawMonitorWait(), JvmtiEnv::RawMonitorNotify(), JvmtiEnv::RawMonitorNotifyAll())

(これらは同名の JVMTI 関数を実現するためのメソッド. (See: [here](no2935uzr.html) for details))




### 詳細(Details)
See: [here](../doxygen/classJvmtiRawMonitor.html) for details

---
## <a name="noy77BYxt0" id="noy77BYxt0">JvmtiPendingMonitors</a>

### 概要(Summary)
JVMTI の関数 (より具体的に言うと, RawMonitor 関係の関数) を実装するためのクラス (See: [here](no2935uzr.html) for details).

まだ JavaThread ができていない状態 (HotSpot が起動中の段階) で RawMonitor が使用された際に, 
それらの RawMonitor を一時的に管理しておくためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

(通常時は RawMonitor は JavaThread 内で管理するが, 
起動中の段階では JavaThread が存在しないため, 特別に管理用のクラスを用意している.
JvmtiPendingMonitors 内に溜められていた JvmtiRawMonitor オブジェクトは,
初期化が終わってメインスレッドを表す JavaThread ができた段階で, その JavaThread へと引き継がれる.)


```
    ((cite: hotspot/src/share/vm/prims/jvmtiRawMonitor.hpp))
    // Onload pending raw monitors
    // Class is used to cache onload or onstart monitor enter
    // which will transition into real monitor when
    // VM is fully initialized.
    class JvmtiPendingMonitors : public AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JvmtiEnv の RawMonitor 関係のメソッド内
  (JvmtiEnv::DestroyRawMonitor(), JvmtiEnv::RawMonitorEnter(), JvmtiEnv::RawMonitorExit())

  これらは同名の JVMTI 関数を実現するためのメソッド (See: [here](no2935uzr.html) for details).

* JvmtiExport::transition_pending_onload_raw_monitors() 内

  これは初期化終了後に JvmtiPendingMonitors 内の JvmtiRawMonitor オブジェクトをメインスレッドに引き継ぐ関数
  (See: [here](no2935uzr.html) for details).

### 内部構造(Internal structure)
内部では GrowableArray<JvmtiRawMonitor*> で JvmtiRawMonitor を管理している.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiRawMonitor.hpp))
      static GrowableArray<JvmtiRawMonitor*> *_monitors; // Cache raw monitor enter
```

なお管理方法は, 登録処理(enter())は GrowableArray に入れるだけ,
登録の解除処理(exit())は GrowableArray から出すだけ, という非常に簡単な方法.
(各 JvmtiRawMonitor の owner を書き換えたりもしない.
 他のスレッドがロックを取りにきて contention が起こる危険性がないのでこの程度で十分なのだと思われる).




### 詳細(Details)
See: [here](../doxygen/classJvmtiPendingMonitors.html) for details

---
