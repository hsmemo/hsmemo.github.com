---
layout: default
title: JvmtiExtensions クラス 
---
[Top](../index.html)

#### JvmtiExtensions クラス 



---
## <a name="noKst7Wq-w" id="noKst7Wq-w">JvmtiExtensions</a>

### 概要(Summary)
JVMTI の関数を実装するために使われているクラス. 
より具体的に言うと, JVMTI の拡張機能
(GetExtensionFunctions(), GetExtensionEvents(), SetExtensionEventCallback()) 
のための関数を納めた名前空間(AllStatic クラス) (See: [here](no2935nLg.html) for details).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiExtensions.hpp))
    // JvmtiExtensions
    //
    // Maintains the list of extension functions and events in this JVMTI
    // implementation. The list of functions and events can be obtained by
    // the profiler using the JVMTI GetExtensionFunctions and
    // GetExtensionEvents functions.
    
    class JvmtiExtensions : public AllStatic {
```

### 使われ方(Usage)
JvmtiEnv クラスの JVMTI 拡張機能用のメソッド内で(のみ)使用されている
(JvmtiEnv::GetExtensionFunctions(), JvmtiEnv::GetExtensionEvents(), JvmtiEnv::SetExtensionEventCallback())
(See: [here](no2935nLg.html) for details).

### 内部構造(Internal structure)
定義されているメソッドは以下の通り.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiExtensions.hpp))
      // register extensions function
      static void register_extensions();
    
      // returns the list of extension functions
      static jvmtiError get_functions(JvmtiEnv* env, jint* extension_count_ptr,
                                      jvmtiExtensionFunctionInfo** extensions);
    
      // returns the list of extension events
      static jvmtiError get_events(JvmtiEnv* env, jint* extension_count_ptr,
                                   jvmtiExtensionEventInfo** extensions);
    
      // sets the callback function for an extension event and enables the event
      static jvmtiError set_event_callback(JvmtiEnv* env, jint extension_event_index,
                                           jvmtiExtensionEvent callback);
```

なお, フィールドとしては以下のフィールド(のみ)が定義されている
(これらは, 拡張機能用の関数／イベントの一覧を格納している配列).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiExtensions.hpp))
      static GrowableArray<jvmtiExtensionFunctionInfo*>* _ext_functions;
      static GrowableArray<jvmtiExtensionEventInfo*>* _ext_events;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiExtensions.html) for details

---
