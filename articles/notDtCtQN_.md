---
layout: default
title: JvmtiUtil クラス及び SafeResourceMark クラス (JvmtiUtil, SafeResourceMark)
---
[Top](../index.html)

#### JvmtiUtil クラス及び SafeResourceMark クラス (JvmtiUtil, SafeResourceMark)

これらは, JVMTI の機能を実装するために使用される補助クラス.


### クラス一覧(class list)

  * [JvmtiUtil](#no3FhljFg9)
  * [SafeResourceMark](#nokAunaxSR)


---
## <a name="no3FhljFg9" id="no3FhljFg9">JvmtiUtil</a>

### 概要(Summary)
JVMTI の機能を実現するためのクラス. 
より具体的に言うと, JVMTI 処理で使用される様々な補助関数を納めた名前空間(AllStatic クラス).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiUtil.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiUtil
    //
    // class for miscellaneous jvmti utility static methods
    //
    
    class JvmtiUtil : AllStatic {
```

### 内部構造(Internal structure)
定義されているメソッドは以下の通り.

```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiUtil.hpp))
      static ResourceArea* single_threaded_resource_area();
    
      static const char* error_name(int num)    { return _error_names[num]; }    // To Do: add range checking
    
      static const bool has_event_capability(jvmtiEvent event_type, const jvmtiCapabilities* capabilities_ptr);
    
      static const bool  event_threaded(int num) {
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiUtil.html) for details

---
## <a name="nokAunaxSR" id="nokAunaxSR">SafeResourceMark</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef JVMTI_TRACE 時にしか使用されない).

特殊な ResourceMark クラス.
普通の ResourceMark クラスと異なり, Thread が生成される前でも使用できる.

(普通の ResourceMark クラスでは, 確保用のメモリ領域として Thread の ResourceArea を使用するため, 
 Thread が生成されるまでは使用できない.
 このクラスの場合, Thread が存在していなければ代わりに JvmtiUtil::single_threaded_resource_area() を使用するため, 
 hread が生成される前でも使用できる.)

```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiUtil.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class SafeResourceMark
    //
    // ResourceMarks that work before threads exist
    //
    
    class SafeResourceMark : public ResourceMark {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている. ただしどの箇所でも #ifdef JVMTI_TRACE 時にしか使用されない.

* EC_TRACE() マクロ内
* JvmtiEventControllerPrivate::trace_changed(JvmtiThreadState *state, jlong now_enabled, jlong changed) 内
* JvmtiEventControllerPrivate::trace_changed(jlong now_enabled, jlong changed) 内
* EVT_TRACE() マクロ内
* EVT_TRIG_TRACE() マクロ内
* JvmtiTrace::initialize() 内




### 詳細(Details)
See: [here](../doxygen/classSafeResourceMark.html) for details

---
