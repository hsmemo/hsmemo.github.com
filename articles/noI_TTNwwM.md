---
layout: default
title: JvmtiEnvThreadState クラス関連のクラス (JvmtiFramePop, JvmtiFramePops, JvmtiEnvThreadState, 及びそれらの補助クラス(VM_GetCurrentLocation))
---
[Top](../index.html)

#### JvmtiEnvThreadState クラス関連のクラス (JvmtiFramePop, JvmtiFramePops, JvmtiEnvThreadState, 及びそれらの補助クラス(VM_GetCurrentLocation))

これらは, JVMTI の機能を実装するために使われているクラス.
より具体的に言うと, "JVMTI environment" (JVMTI 環境) を実現するためのクラス.


### クラス一覧(class list)

  * [JvmtiEnvThreadState](#noSDfqGDAz)
  * [JvmtiFramePops](#nonmtSVVry)
  * [JvmtiFramePop](#noD5mVFjel)
  * [VM_GetCurrentLocation](#noNqSR25tN)


---
## <a name="noSDfqGDAz" id="noSDfqGDAz">JvmtiEnvThreadState</a>

### 概要(Summary)
JVMTI の "JVMTI environment" (JVMTI 環境) を実装するためのクラス.

このクラスは JVMTI connection (JVMTI 接続) 毎の状態の管理を行う.
JVMTI 接続毎の状態の管理は, このクラスだけでなく, JvmtiEnvBase クラスや JvmtiThreadState クラスと連携して行われている.
このクラスはその中で「JVMTI environment 毎に固有でかつスレッド毎に固有」な状態の管理を担当している
(See: [here](no3718uqQ.html) for details).

主に以下の情報を格納している.

  * JVMTI の NotifyFramePop() 関数で指定されたフレーム番号   (See: [here](no2935pZs.html) for details)

  * 現在の実行地点(実行しているメソッド, および bci(bytecode index))
    
    (この情報はブレークポイントイベントやシングルステップイベントの通知に使われる)

  * JVMTI の SetThreadLocalStorage() でセットされたスレッド固有なデータ (See: [here](no2935cPm.html) for details)

  * 現在有効化されているイベント通知の種別

    (正確には, その中で SetEventNotificationMode() 時に対象スレッドが指定されていたもの
     (See: JvmtiEnvThreadEventEnable) (See: [here](no2935C7Z.html) for details))


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvThreadState.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiEnvThreadState
    //
    // 2. Cache of pending frame_pop_events, created by NotifyFramePop
    //    and lazily initialized.
    // 3: Location of last executed instruction, used to filter out duplicate
    //    events due to instruction rewriting.
    
    class JvmtiEnvThreadState : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JvmtiThreadState オブジェクトの _head_env_thread_state フィールドに(のみ)格納されている.

(正確には, このフィールドは JvmtiEnvThreadState の線形リストを格納するフィールド.
JvmtiEnvThreadState オブジェクトは _next フィールドで次の JvmtiEnvThreadState オブジェクトを指せる構造になっている.
その JvmtiThreadState オブジェクト内で生成した JvmtiEnvThreadState オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 生成箇所(where its instances are created)
JvmtiThreadState::add_env() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
JvmtiThreadState::JvmtiThreadState()
-> JvmtiThreadState::add_env()

JvmtiEnv::JvmtiEnv()
-> JvmtiEnvBase::JvmtiEnvBase()
   -> JvmtiEventController::env_initialize()
      -> JvmtiEventControllerPrivate::env_initialize()
         -> JvmtiThreadState::add_env()
```

#### 削除箇所(where its instances are deleted)
JvmtiThreadState::periodic_clean_up() 内で削除されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている. (See: JvmtiGCMarker)

```
JvmtiGCMarker::JvmtiGCMarker()
-> JvmtiEnvBase::check_for_periodic_clean_up()
   -> JvmtiEnvBase::periodic_clean_up()
      -> JvmtiThreadState::periodic_clean_up()
```

### 内部構造(Internal structure)
SetThreadLocalStorage() でセットされたデータは以下のフィールドに格納される. (See: [here](no2935cPm.html) for details)

```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvThreadState.hpp))
      void              *_agent_thread_local_storage_data; // per env and per thread agent allocated data.
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiEnvThreadState.html) for details

---
## <a name="nonmtSVVry" id="nonmtSVVry">JvmtiFramePops</a>

### 概要(Summary)
JvmtiEnvThreadState クラス内で使用される補助クラス
(なお, コメントでは  "Used by              : JvmtiThreadState" となっているが typo だと思われる).

JVMTI の NotifyFramePop() で予約された frame を記録しておくためのクラス (See: [here](no2935pZs.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvThreadState.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiFramePops
    // Used by              : JvmtiThreadState
    // Used by JVMTI methods: none directly.
    //
    // A collection of JvmtiFramePop.
    // It records what frames on a threads stack should post frame_pop events when they're exited.
    //
    
    class JvmtiFramePops : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JvmtiEnvThreadState オブジェクトの _frame_pops フィールドに(のみ)格納されている.

(ただし, オブジェクトの生成は実際に必要になるまで遅延されている)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvThreadState.hpp))
      // Class used to store pending framepops.
      // lazily initialized by get_frame_pops();
      JvmtiFramePops *_frame_pops;
```

#### 生成箇所(where its instances are created)
JvmtiEnvThreadState::get_frame_pops() 内で(のみ)生成されている (= 初めて使用される時まで生成が遅延されている).

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
JvmtiEnvThreadState::is_frame_pop()
-> JvmtiEnvThreadState::get_frame_pops()

JvmtiEventControllerPrivate::set_frame_pop()
-> JvmtiEnvThreadState::get_frame_pops()

JvmtiEventControllerPrivate::clear_frame_pop()
-> JvmtiEnvThreadState::get_frame_pops()

JvmtiEventControllerPrivate::clear_to_frame_pop()
-> JvmtiEnvThreadState::get_frame_pops()
```

### 内部構造(Internal structure)
定義されているフィールドは以下の1つのみ
(この GrowableArray<int> を使って指定された frame の番号を記録している).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvThreadState.hpp))
      GrowableArray<int>* _pops;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiFramePops.html) for details

---
## <a name="noD5mVFjel" id="noD5mVFjel">JvmtiFramePop</a>

### 概要(Summary)
JVMTI の関数 (より具体的に言うと, NotifyFramePop() 関数) を実装するためのクラス (See: [here](no2935pZs.html) for details).

JvmtiFramePops と他のクラスの間で
「NotifyFramePop() で予約された frame 番号」の情報をやりとりする際に使われる ValueObj クラス (See: JvmtiFramePops).

(なおコメントによると, 多少問題があるので rewrite したい, とのこと)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvThreadState.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiFramePop
    // Used by              : JvmtiFramePops
    // Used by JVMTI methods: none directly.
    //
    // Wrapper class for FramePop, used in the JvmtiFramePops class.
    //
    // Two problems: 1) this isn't being used as a ValueObj class, in
    // several places there are constructors for it. 2) It seems like
    // overkill as a means to get an assert and name the geater than
    // operator.  I'm trying to to rewrite everything.
    
    class JvmtiFramePop VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の1つのみ
(このフィールドが frame の番号を示す. 最も古い frame が 0 で新しくなる毎に 1つずつ増えていく.)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvThreadState.hpp))
     private:
      // Frame number counting from BOTTOM (oldest) frame;
      // bottom frame == #0
      int _frame_number;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiFramePop.html) for details

---
## <a name="noNqSR25tN" id="noNqSR25tN">VM_GetCurrentLocation</a>

### 概要(Summary)
JvmtiEnvThreadState クラス内で使用される補助クラス(VM_Operationクラス).

コンストラクタで指定されたスレッドの現在の実行地点
(実行しているメソッド, および bci (bytecode index))を取得する
(これらの情報は stack が walkable でないと取得しにくいので VM_Operation になっている).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvThreadState.cpp))
    class VM_GetCurrentLocation : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEnvThreadState::reset_current_location() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classVM__GetCurrentLocation.html) for details

---
