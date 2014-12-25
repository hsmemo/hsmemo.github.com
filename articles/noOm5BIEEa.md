---
layout: default
title: JvmtiEventController クラス関連のクラス (JvmtiEventEnabled, JvmtiEnvThreadEventEnable, JvmtiThreadEventEnable, JvmtiEnvEventEnable, JvmtiEventController, 及びそれらの補助クラス(VM_EnterInterpOnlyMode, VM_ChangeSingleStep, JvmtiEventControllerPrivate))
---
[Top](../index.html)

#### JvmtiEventController クラス関連のクラス (JvmtiEventEnabled, JvmtiEnvThreadEventEnable, JvmtiThreadEventEnable, JvmtiEnvEventEnable, JvmtiEventController, 及びそれらの補助クラス(VM_EnterInterpOnlyMode, VM_ChangeSingleStep, JvmtiEventControllerPrivate))

これらは, JVMTI の機能を実装するために使われているクラス.
より具体的に言うと, JVMTI のイベント通知機能を実装するためのクラス
(See: [here](no2935C7Z.html) for details).


### クラス一覧(class list)

  * [JvmtiEventController](#nouacYNGoF)
  * [JvmtiEventEnabled](#noC6AeM4PC)
  * [JvmtiEnvThreadEventEnable](#no0FNsxnmg)
  * [JvmtiThreadEventEnable](#nomhZ0ObX1)
  * [JvmtiEnvEventEnable](#noqGAMS-vV)
  * [VM_EnterInterpOnlyMode](#norm7WRxp4)
  * [VM_ChangeSingleStep](#nojGRn2dtm)
  * [JvmtiEventControllerPrivate](#noHOXq4Wik)


---
## <a name="nouacYNGoF" id="nouacYNGoF">JvmtiEventController</a>

### 概要(Summary)
JVMTI のイベント通知設定の変更処理を行うクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).
(See: [here](no2935C7Z.html) for details).

例えば次のような関数が定義されている.

  * イベント通知の有効／無効を切り替える関数
  * コールバック関数を変更する関数
  * watchpoint の位置を変更する関数
  * framepop 設定を変更する関数
  * etc


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // JvmtiEventController
    //
    // The class is the access point for all actions that change
    // which events are active, this include:
    //      enabling and disabling events
    //      changing the callbacks/eventhook (they may be null)
    //      setting and clearing field watchpoints
    //      setting frame pops
    //      encountering frame pops
    //
    // for inlines see jvmtiEventController_inline.hpp
    //
    
    class JvmtiEventController : AllStatic {
```

### 内部構造(Internal structure)
定義されているメソッドは以下の通り.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
      static bool is_enabled(jvmtiEvent event_type);
    
      // events that can ONLY be enabled/disabled globally (can't toggle on individual threads).
      static bool is_global_event(jvmtiEvent event_type);
    
      // is the event_type valid?
      // to do: check against valid event array
      static bool is_valid_event_type(jvmtiEvent event_type) {
        return ((int)event_type >= TOTAL_MIN_EVENT_TYPE_VAL)
            && ((int)event_type <= TOTAL_MAX_EVENT_TYPE_VAL);
      }
    
      // Use (thread == NULL) to enable/disable an event globally.
      // Use (thread != NULL) to enable/disable an event for a particular thread.
      // thread is ignored for events that can only be specified globally
      static void set_user_enabled(JvmtiEnvBase *env, JavaThread *thread,
                                   jvmtiEvent event_type, bool enabled);
    
      // Setting callbacks changes computed enablement and must be done
      // at a safepoint otherwise a NULL callback could be attempted
      static void set_event_callbacks(JvmtiEnvBase *env,
                                      const jvmtiEventCallbacks* callbacks,
                                      jint size_of_callbacks);
    
      // Sets the callback function for a single extension event and enables
      // (or disables it).
      static void set_extension_event_callback(JvmtiEnvBase* env,
                                               jint extension_event_index,
                                               jvmtiExtensionEvent callback);
    
      static void set_frame_pop(JvmtiEnvThreadState *env_thread, JvmtiFramePop fpop);
      static void clear_frame_pop(JvmtiEnvThreadState *env_thread, JvmtiFramePop fpop);
      static void clear_to_frame_pop(JvmtiEnvThreadState *env_thread, JvmtiFramePop fpop);
    
      static void change_field_watch(jvmtiEvent event_type, bool added);
    
      static void thread_started(JavaThread *thread);
      static void thread_ended(JavaThread *thread);
    
      static void env_initialize(JvmtiEnvBase *env);
      static void env_dispose(JvmtiEnvBase *env);
    
      static void vm_start();
      static void vm_init();
      static void vm_death();
```

### 備考(Notes)
なお, フィールドとしては以下のフィールド(のみ)が定義されている. 
これは他の箇所の JvmtiEventEnabled 設定をキャッシュしておくためのもの (See: [here](no2935C7Z.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
      // for all environments, global array indexed by jvmtiEvent
      static JvmtiEventEnabled _universal_global_event_enabled;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiEventController.html) for details

---
## <a name="noC6AeM4PC" id="noC6AeM4PC">JvmtiEventEnabled</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するためのクラス.

現在有効化されているイベント通知の種別を表す ValueObj クラス (See: [here](no2935C7Z.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // JvmtiEventEnabled
    //
    // Utility class
    //
    // A boolean array indexed by event_type, used as an internal
    // data structure to track what JVMTI event types are enabled.
    // Used for user set enabling and disabling (globally and on a
    // per thread basis), and for computed merges across environments,
    // threads and the VM as a whole.
    //
    // for inlines see jvmtiEventController_inline.hpp
    //
    
    class JvmtiEventEnabled VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

(4 つのクラスに分散させることで, 有効化されているイベント通知を
env 毎およびスレッド毎(正確には スレッドと env の組み合わせごと) に分けて管理している. (See: [here](no2935C7Z.html) for details))

* JvmtiEventController クラス

    * JvmtiEventController クラスの _universal_global_event_enabled フィールド (static フィールド)
      
      前回の JvmtiEventControllerPrivate::recompute_enabled() で計算した結果がキャッシュされている模様.

      (以下の JvmtiEnvEventEnable, JvmtiThreadEventEnable, JvmtiEnvThreadEventEnable の enalbed の和集合.
      全体として enabled になるべきイベントは最終的にはこれで決まる)

* JvmtiEnvEventEnable クラス

    * 各 JvmtiEnvEventEnable オブジェクトの _event_user_enabled フィールド
      
      ユーザーが SetEventNotificationMode() でイベント通知を有効化したイベント
      (の内で, SetEventNotificationMode で対象が全スレッドと指定されたもの)

    * 各 JvmtiEnvEventEnable オブジェクトの _event_callback_enabled フィールド
      
      ユーザーが SetEventCallbacks() や SetExtensionEventCallback() でコールバックを設定しているイベント一覧.
      (See: set_event_callbacks(), set_extension_event_callback())

    * 各 JvmtiEnvEventEnable オブジェクトの _event_enabled フィールド
      
      前回の JvmtiEventControllerPrivate::recompute_env_enabled() で計算した結果がキャッシュされている模様.

* JvmtiThreadEventEnable クラス
  
    * 各 JvmtiThreadEventEnable オブジェクトの _event_enabled フィールド
      
      前回の JvmtiEventControllerPrivate::recompute_thread_enabled() で計算した結果がキャッシュされている模様
      (その thread が関連する全部の JvmtiEnvThreadEventEnable の enabled の和集合).

* JvmtiEnvThreadEventEnable クラス

    * 各 JvmtiEnvThreadEventEnable オブジェクトの _event_user_enabled フィールド
      
      ユーザーが SetEventNotificationMode() でイベント通知を有効化したイベント
      (の内で, SetEventNotificationMode で対象スレッドが指定されていたもの)

    * 各 JvmtiEnvThreadEventEnable オブジェクトの _event_enabled フィールド
      
      前回の JvmtiEventControllerPrivate::recompute_env_thread_enabled() で計算した結果がキャッシュされている模様.

### 内部構造(Internal structure)
定義されているフィールドは以下の1つのみ (デバッグ時には 2つ).

なお, _enabled_bits フィールドは bit mask として使用されている. 
この bit mask は「どのイベント通知種別が有効化されているか」を示す
(no2935C7Z)).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
      jlong _enabled_bits;
    #ifndef PRODUCT
      enum {
        JEE_INIT_GUARD = 0xEAD0
      } _init_guard;
    #endif
```

### 備考(Notes)
「インスタンスの格納場所」に記述したとおり, 
ユーザーが SetEventNotificationMode()/SetEventCallbacks()/SetExtensionEventCallback()
で指定した内容を直接的に格納している JvmtiEventEnabled オブジェクトと,
それを元に計算した結果をキャッシュしているだけの JvmtiEventEnabled オブジェクトがある.

キャッシュしているのは, 前回と今回で変化が有ったかどうかを調べるため (<= 結局, 高速化のため?? #TODO).

なお, ユーザーの指定内容を直接的に格納しているのは以下の3つ (その他は全てキャッシュ).

* 各 JvmtiEnvEventEnable オブジェクトの _event_user_enabled フィールド
* 各 JvmtiEnvEventEnable オブジェクトの _event_callback_enabled フィールド
* 各 JvmtiEnvThreadEventEnable オブジェクトの _event_user_enabled フィールド)




### 詳細(Details)
See: [here](../doxygen/classJvmtiEventEnabled.html) for details

---
## <a name="no0FNsxnmg" id="no0FNsxnmg">JvmtiEnvThreadEventEnable</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するためのクラス.

現在有効化されているイベント通知の種別を管理するクラス. 
有効化されているイベント通知種別の管理は, このクラスだけでなく, 
JvmtiEnvEventEnable クラスや JvmtiThreadEventEnable クラスと連携して行われている. 
このクラスはその中で「JVMTI environment 毎に固有でかつスレッド毎に固有」なイベント通知設定の管理を担当している
(See: [here](no2935C7Z.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // JvmtiEnvThreadEventEnable
    //
    // JvmtiEventController data specific to a particular environment and thread.
    //
    // for inlines see jvmtiEventController_inline.hpp
    //
    
    class JvmtiEnvThreadEventEnable VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JvmtiEnvThreadState オブジェクトの _event_enable フィールドに(のみ)格納されている (See: JvmtiEnvThreadState).

### 内部構造(Internal structure)
定義されているフィールドは以下のもののみ (See: JvmtiEventEnabled).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
      JvmtiEventEnabled _event_user_enabled;
      JvmtiEventEnabled _event_enabled;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiEnvThreadEventEnable.html) for details

---
## <a name="nomhZ0ObX1" id="nomhZ0ObX1">JvmtiThreadEventEnable</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するためのクラス.

現在有効化されているイベント通知の種別を管理するクラス. 
有効化されているイベント通知種別の管理は, このクラスだけでなく, 
JvmtiEnvEventEnable クラスや JvmtiEnvThreadEventEnable クラスと連携して行われている. 
このクラスはその中で「スレッド毎に固有」なイベント通知設定の管理を担当している
(See: [here](no2935C7Z.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // JvmtiThreadEventEnable
    //
    // JvmtiEventController data specific to a particular thread.
    //
    // for inlines see jvmtiEventController_inline.hpp
    //
    
    class JvmtiThreadEventEnable VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JvmtiThreadState オブジェクトの _thread_event_enable フィールドに(のみ)格納されている (See: JvmtiThreadState).

### 内部構造(Internal structure)
定義されているフィールドは以下のもののみ (See: JvmtiEventEnabled).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
      JvmtiEventEnabled _event_enabled;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiThreadEventEnable.html) for details

---
## <a name="noqGAMS-vV" id="noqGAMS-vV">JvmtiEnvEventEnable</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するためのクラス.

現在有効化されているイベント通知の種別を管理するクラス. 
有効化されているイベント通知種別の管理は, このクラスだけでなく, 
JvmtiThreadEventEnable クラスや JvmtiEnvThreadEventEnable クラスと連携して行われている. 
このクラスはその中で「JVMTI environment 毎に固有」なイベント通知設定の管理を担当している
(See: [here](no2935C7Z.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // JvmtiEnvEventEnable
    //
    // JvmtiEventController data specific to a particular environment.
    //
    // for inlines see jvmtiEventController_inline.hpp
    //
    
    class JvmtiEnvEventEnable VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JvmtiEnvBase オブジェクトの _env_event_enable フィールドに(のみ)格納されている (See: JvmtiEnvBase).

### 内部構造(Internal structure)
定義されているフィールドは以下のもののみ (See: JvmtiEventEnabled).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.hpp))
      // user set global event enablement indexed by jvmtiEvent
      JvmtiEventEnabled _event_user_enabled;
    
      // this flag indicates the presence (true) or absence (false) of event callbacks
      // it is indexed by jvmtiEvent
      JvmtiEventEnabled _event_callback_enabled;
    
      // indexed by jvmtiEvent true if enabled globally or on any thread.
      // True only if there is a callback for it.
      JvmtiEventEnabled _event_enabled;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiEnvEventEnable.html) for details

---
## <a name="norm7WRxp4" id="norm7WRxp4">VM_EnterInterpOnlyMode</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するためのクラス.

HotSpot の実行状態を interp_only_mode に遷移させるための VM_Operation クラス (See: [here](no3059eFS.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.cpp))
    ///////////////////////////////////////////////////////////////
    //
    // VM_EnterInterpOnlyMode
    //
    
    class VM_EnterInterpOnlyMode : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEventControllerPrivate::enter_interp_only_mode() 内で(のみ)使用されている (See: [here](no2935C7Z.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__EnterInterpOnlyMode.html) for details

---
## <a name="nojGRn2dtm" id="nojGRn2dtm">VM_ChangeSingleStep</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するためのクラス.

SingleStep イベントを実現するための VM_Operation クラス.
HotSpot の実行状態を SingleStep モードに遷移させる.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.cpp))
    ///////////////////////////////////////////////////////////////
    //
    // VM_ChangeSingleStep
    //
    
    class VM_ChangeSingleStep : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEventControllerPrivate::recompute_enabled() 内で(のみ)使用されている (See: [here](no2935C7Z.html) and [here](no7882EDP.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__ChangeSingleStep.html) for details

---
## <a name="noHOXq4Wik" id="noHOXq4Wik">JvmtiEventControllerPrivate</a>

### 概要(Summary)
JvmtiEventController クラス内で使用される補助クラス.

JvmtiEventController クラス用の補助関数を納めた名前空間(AllStatic クラス) (See: [here](no2935C7Z.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.cpp))
    ///////////////////////////////////////////////////////////////
    //
    // JvmtiEventControllerPrivate
    //
    // Private internal implementation methods for JvmtiEventController.
    //
    // These methods are thread safe either because they are called
    // in early VM initialization which is single threaded, or they
    // hold the JvmtiThreadState_lock.
    //
    
    class JvmtiEventControllerPrivate : public AllStatic {
```

### 使われ方(Usage)
JvmtiEventController クラスの各種メソッド内で(のみ)使用されている (See: [here](no2935C7Z.html) for details).




### 詳細(Details)
See: [here](../doxygen/classJvmtiEventControllerPrivate.html) for details

---
