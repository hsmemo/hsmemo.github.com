---
layout: default
title: 各種の JVMTI の処理で使われるクラス (GrowableElement, GrowableCache, JvmtiBreakpointCache, JvmtiBreakpoint, VM_ChangeBreakpoints, JvmtiBreakpoints, JvmtiCurrentBreakpoints, VM_GetOrSetLocal, VM_GetReceiver, JvmtiSuspendControl, JvmtiDeferredEvent, JvmtiDeferredEventQueue, JvmtiDeferredEventQueue::QueueNode)
---
[Top](../index.html)

#### 各種の JVMTI の処理で使われるクラス (GrowableElement, GrowableCache, JvmtiBreakpointCache, JvmtiBreakpoint, VM_ChangeBreakpoints, JvmtiBreakpoints, JvmtiCurrentBreakpoints, VM_GetOrSetLocal, VM_GetReceiver, JvmtiSuspendControl, JvmtiDeferredEvent, JvmtiDeferredEventQueue, JvmtiDeferredEventQueue::QueueNode)

これらは, JVMTI の機能を実装するために使われているクラス.


### クラス一覧(class list)

  * [GrowableElement](#nokEKh5PSZ)
  * [GrowableCache](#no8SKP5mx1)
  * [JvmtiBreakpointCache](#no09e_mczx)
  * [JvmtiBreakpoint](#nonJjJpbq9)
  * [VM_ChangeBreakpoints](#nopXOv2viB)
  * [JvmtiBreakpoints](#noyodSBMnC)
  * [JvmtiCurrentBreakpoints](#no19oDKk7E)
  * [VM_GetOrSetLocal](#noW-vIuvjb)
  * [VM_GetReceiver](#no9IJVKg0l)
  * [JvmtiSuspendControl](#nokEMX7k-2)
  * [JvmtiDeferredEvent](#no8BOAw7tx)
  * [JvmtiDeferredEventQueue](#noKQV76aCM)
  * [JvmtiDeferredEventQueue::QueueNode](#noC6sQ3AVb)


---
## <a name="nokEKh5PSZ" id="nokEKh5PSZ">GrowableElement</a>

### 概要(Summary)
JVMTI のブレークポイント処理で使用される補助クラス (See: [here](no3718uV0.html) for details).

ブレークポイント情報を表すクラスの基底クラス.
GrowableCache クラス内に格納されて管理される.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス (See: JvmtiBreakpoint).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class GrowableCache, GrowableElement
    // Used by              : JvmtiBreakpointCache
    // Used by JVMTI methods: none directly.
    //
    // GrowableCache is a permanent CHeap growable array of <GrowableElement *>
    //
    // In addition, the GrowableCache maintains a NULL terminated cache array of type address
    // that's created from the element array using the function:
    //     address GrowableElement::getCacheValue().
    //
    // Whenever the GrowableArray changes size, the cache array gets recomputed into a new C_HEAP allocated
    // block of memory. Additionally, every time the cache changes its position in memory, the
    //    void (*_listener_fun)(void *this_obj, address* cache)
    // gets called with the cache's new address. This gives the user of the GrowableCache a callback
    // to update its pointer to the address cache.
    //
```


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    class GrowableElement : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classGrowableElement.html) for details

---
## <a name="no8SKP5mx1" id="no8SKP5mx1">GrowableCache</a>

### 概要(Summary)
JVMTI のブレークポイント処理で使用される補助クラス (See: [here](no3718uV0.html) for details).

ブレークポイント情報を管理するための可変長の配列.
この中にブレークポイント情報を表す GrowableElement オブジェクト
(実際には GrowableElement は abstract class なので, そのサブクラスである JvmtiBreakpoint オブジェクト)
が格納される.

なお, GrowableCache オブジェクト内には,
GrowableElement オブジェクト自体の配列とは別に,
_cache という address 配列も保持している.
これは, 配列中の GrowableElement オブジェクトが
GrowableElement::getCacheValue() の呼び出しに対して返した結果を集めたもの.

また, GrowableElement オブジェクトの数が増減すると _cache も C ヒープ状の別の場所に確保し直されるが,
外部から _cache を参照している場合のために,
_cache のアドレスが変わった場合に呼び出されるコールバックを設定しておくことができる.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class GrowableCache, GrowableElement
    // Used by              : JvmtiBreakpointCache
    // Used by JVMTI methods: none directly.
    //
    // GrowableCache is a permanent CHeap growable array of <GrowableElement *>
    //
    // In addition, the GrowableCache maintains a NULL terminated cache array of type address
    // that's created from the element array using the function:
    //     address GrowableElement::getCacheValue().
    //
    // Whenever the GrowableArray changes size, the cache array gets recomputed into a new C_HEAP allocated
    // block of memory. Additionally, every time the cache changes its position in memory, the
    //    void (*_listener_fun)(void *this_obj, address* cache)
    // gets called with the cache's new address. This gives the user of the GrowableCache a callback
    // to update its pointer to the address cache.
    //
```


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    class GrowableCache VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JvmtiBreakpointCache オブジェクトの _cache フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(JvmtiBreakpointCache クラスの _cache フィールドは, ポインタ型ではなく実体なので,
 JvmtiBreakpointCache オブジェクトの生成時に一緒に生成される)

### 内部構造(Internal structure)

```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      // Parallel array of cached values
      address *_cache;
```

### 備考(Notes)
なお, 現在の JvmtiBreakpoint::getCacheValue() の実装は JvmtiBreakpoint::getBcp() を返すようになっている模様.

```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      address getCacheValue()         { return getBcp(); }
```

そして JvmtiBreakpoint::getBcp() の方は, 
methodOop 内のブレークポイントに該当するアドレスを返すようになっている模様.

```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.cpp))
    address JvmtiBreakpoint::getBcp() {
      return _method->bcp_from(_bci);
    }
```

また, コールバック(_listener_fun)に設定されるのは,
現状では JvmtiCurrentBreakpoints::listener_fun() だけである模様.
設定される処理パスは以下の通り.

```
JvmtiCurrentBreakpoints::get_jvmti_breakpoints()
-> JvmtiBreakpoints::JvmtiBreakpoints()
   -> JvmtiBreakpointCache::initialize()
      -> GrowableCache::initialize()
```




### 詳細(Details)
See: [here](../doxygen/classGrowableCache.html) for details

---
## <a name="no09e_mczx" id="no09e_mczx">JvmtiBreakpointCache</a>

### 概要(Summary)
JVMTI のブレークポイント処理で使用される補助クラス (See: [here](no3718uV0.html) for details).

ブレークポイント情報を管理するための可変長の配列.
実体としては, 単なる GrowableCache のラッパー.
GrowableCache との唯一の違いは, 
GrowableCache だと JvmtiBreakpoint オブジェクトを入れるには GrowableElement にキャストしないといけないが, 
JvmtiBreakpointCache だと内部的にキャストしてくれるのでキャストの必要が無い, という点.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiBreakpointCache
    // Used by              : JvmtiBreakpoints
    // Used by JVMTI methods: none directly.
    // Note   : typesafe wrapper for GrowableCache of JvmtiBreakpoint
    //
    
    class JvmtiBreakpointCache : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JvmtiBreakpoints オブジェクトの _bps フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(JvmtiBreakpoints クラスの _bps フィールドは, ポインタ型ではなく実体なので,
 JvmtiBreakpoints オブジェクトの生成時に一緒に生成される)



### 詳細(Details)
See: [here](../doxygen/classJvmtiBreakpointCache.html) for details

---
## <a name="nonJjJpbq9" id="nonJjJpbq9">JvmtiBreakpoint</a>

### 概要(Summary)
JVMTI のブレークポイント処理で使用される補助クラス (See: [here](no3718uV0.html) for details).
GrowableElement クラスの具象サブクラス (なお, 現在はこのクラスが唯一のサブクラス).

JVMTI の SetBreakpoint() によって設定されたブレークポイントを表す.
1つのブレークポイントに付き1つの JvmtiBreakpoint オブジェクトが生成される.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiBreakpoint
    // Used by              : JvmtiBreakpoints
    // Used by JVMTI methods: SetBreakpoint, ClearBreakpoint, ClearAllBreakpoints
    // Note: Extends GrowableElement for use in a GrowableCache
    //
    // A JvmtiBreakpoint describes a location (class, method, bci) to break at.
    //
```


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    class JvmtiBreakpoint : public GrowableElement {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 GrowableCache オブジェクト内の _elements フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
JvmtiEnv::SetBreakpoint() 内の局所変数として生成された後,
SetBreakpoint() の処理中で呼び出される JvmtiBreakpoint::clone() によって
C ヒープ上に生成される (See: [here](no3718uV0.html) for details).




### 詳細(Details)
See: [here](../doxygen/classJvmtiBreakpoint.html) for details

---
## <a name="nopXOv2viB" id="nopXOv2viB">VM_ChangeBreakpoints</a>

### 概要(Summary)
JVMTI のブレークポイント処理で使用される補助クラス(VM_Operationクラス) (See: [here](no3718uV0.html) for details).

ブレークポイントの追加や削除を行う
(ブレークポイントの変更は非同期に行うと危険なので VM_Operation となっている).

```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class VM_ChangeBreakpoints
    // Used by              : JvmtiBreakpoints
    // Used by JVMTI methods: none directly.
    // Note: A Helper class.
    //
    // VM_ChangeBreakpoints implements a VM_Operation for ALL modifications to the JvmtiBreakpoints class.
    //
    
    class VM_ChangeBreakpoints : public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no3718uV0.html) for details).

* JvmtiBreakpoints::set()
* JvmtiBreakpoints::clear()
* JvmtiBreakpoints::clearall()




### 詳細(Details)
See: [here](../doxygen/classVM__ChangeBreakpoints.html) for details

---
## <a name="noyodSBMnC" id="noyodSBMnC">JvmtiBreakpoints</a>

### 概要(Summary)
JVMTI のブレークポイント処理で使用される補助クラス (See: [here](no3718uV0.html) for details).

ブレークポイント情報を管理するための可変長の配列.
実体としては, 単なるJvmtiBreakpointCacheのラッパー+α, といった感じ.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiBreakpoints
    // Used by              : JvmtiCurrentBreakpoints
    // Used by JVMTI methods: none directly
    // Note: A Helper class
    //
    // JvmtiBreakpoints is a GrowableCache of JvmtiBreakpoint.
    // All changes to the GrowableCache occur at a safepoint using VM_ChangeBreakpoints.
    //
    // Because _bps is only modified at safepoints, its possible to always use the
    // cached byte code pointers from _bps without doing any synchronization (see JvmtiCurrentBreakpoints).
    //
    // It would be possible to make JvmtiBreakpoints a static class, but I've made it
    // CHeap allocated to emphasize its similarity to JvmtiFramePops.
    //
    
    class JvmtiBreakpoints : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
JvmtiCurrentBreakpoints クラス内の _jvmti_breakpoints フィールドに(のみ)格納されている
(ただし, JvmtiBreakpoints オブジェクトの生成自体は実際に必要になるまで遅延されている).

#### 生成箇所(where its instances are created)
JvmtiCurrentBreakpoints::get_jvmti_breakpoints() 内で(のみ)生成されている
(= 初めて使用される時まで生成が遅延されている) (See: [here](no3718uV0.html) for details).

### 内部構造(Internal structure)
定義されているフィールドは以下のもののみ.

```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      JvmtiBreakpointCache _bps;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiBreakpoints.html) for details

---
## <a name="no19oDKk7E" id="no19oDKk7E">JvmtiCurrentBreakpoints</a>

### 概要(Summary)
JVMTI のブレークポイント機能の処理を行うクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)) (See: [here](no3718uV0.html) for details).

以下のような機能を提供している.

  * 指定箇所の命令がブレークポイント命令かどうかを判定する機能 (is_breakpoint)
  * JvmtiBreakpoints を, 必要になるまで生成しない(生成を遅延する)機能
  * ブレークポイント関係のクラスが保持しているポインタに関する oops_do() 処理 (Garbage Collection を支援する機能)

```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiCurrentBreakpoints
    //
    // A static wrapper class for the JvmtiBreakpoints that provides:
    // 1. a fast inlined function to check if a byte code pointer is a breakpoint (is_breakpoint).
    // 2. a function for lazily creating the JvmtiBreakpoints class (this is not strictly necessary,
    //    but I'm copying the code from JvmtiThreadState which needs to lazily initialize
    //    JvmtiFramePops).
    // 3. An oops_do entry point for GC'ing the breakpoint array.
    //
    
    class JvmtiCurrentBreakpoints : public AllStatic {
```

### 内部構造(Internal structure)
定義されている public メソッドは, 以下の通り.

  * get_jvmti_breakpoints()

    JvmtiBreakpoints オブジェクトを返す. 
    (JvmtiBreakpoints オブジェクトはこのメソッドが初回に呼ばれた際に生成する(遅延生成))

  * is_breakpoint()

	指定箇所の命令がブレークポイント命令かどうかを判定する.

  * oops_do(), gc_epilogue()

    Garbage Collection 処理用の補助関数.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      static void initialize();
      static void destroy();
    
      // lazily create _jvmti_breakpoints and _breakpoint_list
      static JvmtiBreakpoints& get_jvmti_breakpoints();
    
      // quickly test whether the bcp matches a cached breakpoint in the list
      static inline bool is_breakpoint(address bcp);
    
      static void oops_do(OopClosure* f);
      static void gc_epilogue();
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiCurrentBreakpoints.html) for details

---
## <a name="noW-vIuvjb" id="noW-vIuvjb">VM_GetOrSetLocal</a>

### 概要(Summary)
JVMTI の関数 (より具体的に言うと, 局所変数にアクセスする JVMTI 関数のうちで GetLocalInstance() 以外のもの) 
を実装するための補助クラス(VM_Operationクラス)
(See: [here](no2935GIU.html) for details).

該当する JVMTI 関数は以下の通り.

* GetLocalObject()
* GetLocalInt()
* GetLocalLong()
* GetLocalFloat()
* GetLocalDouble()
* SetLocalObject()
* SetLocalInt()
* SetLocalLong()
* SetLocalFloat()
* SetLocalDouble()
   
(なお, VM_Operation のサブクラスになっているのは, 
 アクセス先が interpreter frame の場合には oop map へのアクセスが必要であり, 
 それには VM Thread から行わないと危険なため, とのこと.
 
 ただし 1.5 以降では oop map もロックで保護されているので, 
 VM_Operation にする代わりに, 
 アクセス先のスレッドを suspend させてロックも取得してからアクセスするという実装でもよいかもしれない, とのこと.)

```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    ///////////////////////////////////////////////////////////////
    // The get/set local operations must only be done by the VM thread
    // because the interpreter version needs to access oop maps, which can
    // only safely be done by the VM thread
    //
    // I'm told that in 1.5 oop maps are now protected by a lock and
    // we could get rid of the VM op
    // However if the VM op is removed then the target thread must
    // be suspended AND a lock will be needed to prevent concurrent
    // setting of locals to the same java thread. This lock is needed
    // to prevent compiledVFrames from trying to add deferred updates
    // to the thread simultaneously.
    //
    class VM_GetOrSetLocal : public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no2935GIU.html) for details).

* JvmtiEnv::GetLocalObject()
* JvmtiEnv::GetLocalInt()
* JvmtiEnv::GetLocalLong()
* JvmtiEnv::GetLocalFloat()
* JvmtiEnv::GetLocalDouble()
* JvmtiEnv::SetLocalObject()
* JvmtiEnv::SetLocalInt()
* JvmtiEnv::SetLocalLong()
* JvmtiEnv::SetLocalFloat()
* JvmtiEnv::SetLocalDouble()




### 詳細(Details)
See: [here](../doxygen/classVM__GetOrSetLocal.html) for details

---
## <a name="no9IJVKg0l" id="no9IJVKg0l">VM_GetReceiver</a>

### 概要(Summary)
JVMTI の関数 (より具体的に言うと GetLocalInstance() 関数) を実装するための補助クラス(VM_Operationクラス)
(See: [here](no2935GIU.html) for details).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    class VM_GetReceiver : public VM_GetOrSetLocal {
```

### 使われ方(Usage)
JvmtiEnv::GetLocalInstance() 内で(のみ)使用されている (See: [here](no2935GIU.html) for details).

### 内部構造(Internal structure)
スーパークラスである VM_GetOrSetLocal との違いは getting_receiver() メソッドの返値だけ.
これは VM_GetOrSetLocal::doit_prologue() 内の挙動に影響する (See: [here](no2935GIU.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GetReceiver.html) for details

---
## <a name="nokEMX7k-2" id="nokEMX7k-2">JvmtiSuspendControl</a>

### 概要(Summary)
JVMTI の関数 (より具体的に言うと, スレッドの suspend/resume 処理を行う関数) を実装するためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)) (See: [here](no2935HQP.html) for details).

該当する JVMTI 関数は以下の通り.

* SuspendThread()
* SuspendThreadList()
* ResumeThread()
* ResumeThreadList()


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    ///////////////////////////////////////////////////////////////
    //
    // class JvmtiSuspendControl
    //
    // Convenience routines for suspending and resuming threads.
    //
    // All attempts by JVMTI to suspend and resume threads must go through the
    // JvmtiSuspendControl interface.
    //
    // methods return true if successful
    //
    class JvmtiSuspendControl : public AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている

* JvmtiEnv::SuspendThread()

  (See: [here](no2935HQP.html) for details).

* JvmtiEnv::SuspendThreadList()

  (See: [here](no2935HQP.html) for details).

* JvmtiEnv::ResumeThread()

  (See: [here](no2935HQP.html) for details).

* JvmtiEnv::ResumeThreadList()

  (See: [here](no2935HQP.html) for details).

* JvmtiEnv::NotifyFramePop()

  (ただし, TraceJVMTICalls オプションが指定されている際にしか使われていない.
   使われ方も JvmtiSuspendControl::print() でトレース出力を出すだけ.
   (See: [here](no2935pZs.html) for details))

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      // suspend the thread, taking it to a safepoint
      static bool suspend(JavaThread *java_thread);
      // resume the thread
      static bool resume(JavaThread *java_thread);
    
      static void print();
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiSuspendControl.html) for details

---
## <a name="no8BOAw7tx" id="no8BOAw7tx">JvmtiDeferredEvent</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するための補助クラス(ValueObjクラス).
より具体的に言うと, イベント通知の遅延処理を実現するためのクラス
(See: [here](no3718UPQ.html) for details).

遅延したいイベントの内容を表す.
1つの JvmtiDeferredEvent オブジェクトが 1つのイベント通知に対応する.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    class JvmtiDeferredEvent VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
JvmtiDeferredEventQueue クラス内(の JvmtiDeferredEventQueue::QueueNode オブジェクト内)に(のみ)格納されている.

#### 生成箇所(where its instances are created)
(ValueObj クラスなので「生成」というのは少し違和感があるが)
以下のファクトリメソッドが用意されており, その中で(のみ)生成されている.

* JvmtiDeferredEvent::compiled_method_load_event()
* JvmtiDeferredEvent::compiled_method_unload_event()
* JvmtiDeferredEvent::dynamic_code_generated_event()

そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている (See: [here](no3718UPQ.html) for details).

```
ciEnv::register_method()
-> nmethod::post_compiled_method_load_event()
   -> JvmtiDeferredEvent::compiled_method_load_event()

AdapterHandlerLibrary::create_native_wrapper()
-> nmethod::post_compiled_method_load_event()
   -> JvmtiDeferredEvent::compiled_method_load_event()

nmethod::make_unloaded()
-> nmethod::post_compiled_method_unload()
   -> JvmtiDeferredEvent::compiled_method_unload_event()

nmethod::make_not_entrant_or_zombie()
-> nmethod::post_compiled_method_unload()
   -> JvmtiDeferredEvent::compiled_method_unload_event()

JvmtiExport::post_dynamic_code_generated()
-> JvmtiDeferredEvent::dynamic_code_generated_event()
```

### 内部構造(Internal structure)
定義されているフィールドは以下のもののみ.

```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      Type _type;
      union {
        nmethod* compiled_method_load;
        struct {
          nmethod* nm;
          jmethodID method_id;
          const void* code_begin;
        } compiled_method_unload;
        struct {
          const char* name;
          const void* code_begin;
          const void* code_end;
        } dynamic_code_generated;
      } _event_data;
```

(なお, Type 型は以下のような enum 型)


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      typedef enum {
        TYPE_NONE,
        TYPE_COMPILED_METHOD_LOAD,
        TYPE_COMPILED_METHOD_UNLOAD,
        TYPE_DYNAMIC_CODE_GENERATED
      } Type;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiDeferredEvent.html) for details

---
## <a name="noKQV76aCM" id="noKQV76aCM">JvmtiDeferredEventQueue</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するための補助クラス.
より具体的に言うと, イベント通知の遅延処理を実現するためのクラス
(See: [here](no3718UPQ.html) for details).

遅延通知するイベントを溜めておくためのキューを表すクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    /**
     * Events enqueued on this queue wake up the Service thread which dequeues
     * and posts the events.  The Service_lock is required to be held
     * when operating on the queue (except for the "pending" events).
     */
    class JvmtiDeferredEventQueue : AllStatic {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
イベントを追加するには以下の２通りの方法がある.

* Service_lock を保持した状態で JvmtiDeferredEventQueue::enqueue() を呼び出す.
* (ロックにかかわらず) JvmtiDeferredEventQueue::add_pending_event() を呼び出す.

追加したイベントを取り出すには, JvmtiDeferredEventQueue::dequeue() を呼び出せばいい.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている (See: [here](no3718UPQ.html) for details).

* nmethod::post_compiled_method_load_event()
* nmethod::post_compiled_method_unload()
* JvmtiExport::post_dynamic_code_generated()
* ServiceThread::service_thread_entry()

### 内部構造(Internal structure)
フィールドとしては以下のフィールド(のみ)が定義されている


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      static QueueNode* _queue_head;             // Hold Service_lock to access
      static QueueNode* _queue_tail;             // Hold Service_lock to access
      static volatile QueueNode* _pending_list;  // Uses CAS for read/update
```

そして, 定義されている public メソッドは以下の通り.

(なお, JvmtiDeferredEventQueue::enqueue() と JvmtiDeferredEventQueue::dequeue() は
_queue_head と _queue_tail を操作する.
JvmtiDeferredEventQueue::add_pending_event() は _pending_list に追加を行う.)


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      // Must be holding Service_lock when calling these
      static bool has_events() KERNEL_RETURN_(false);
      static void enqueue(const JvmtiDeferredEvent& event) KERNEL_RETURN;
      static JvmtiDeferredEvent dequeue() KERNEL_RETURN_(JvmtiDeferredEvent());
    
      // Used to enqueue events without using a lock, for times (such as during
      // safepoint) when we can't or don't want to lock the Service_lock.
      //
      // Events will be held off to the side until there's a call to
      // dequeue(), enqueue(), or process_pending_events() (all of which require
      // the holding of the Service_lock), and will be enqueued at that time.
      static void add_pending_event(const JvmtiDeferredEvent&) KERNEL_RETURN;
```

(なお, JvmtiDeferredEventQueue::add_pending_event() によって 
_pending_list に追加された要素を _queue_head/_queue_tail に移動する作業は, 
以下の private メソッドによって行われている)


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      // Transfers events from the _pending_list to the _queue.
      static void process_pending_events() KERNEL_RETURN;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiDeferredEventQueue.html) for details

---
## <a name="noC6sQ3AVb" id="noC6sQ3AVb">JvmtiDeferredEventQueue::QueueNode</a>

### 概要(Summary)
JvmtiDeferredEventQueue クラス内で使用される補助クラス.

遅延したいイベントを線形リスト状にして溜めておくために使用される (See: [here](no3718UPQ.html) for details).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
      class QueueNode : public CHeapObj {
```

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む (そして, メソッドはこれらのフィールドへのアクセサメソッドのみ).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
        JvmtiDeferredEvent _event;
        QueueNode* _next;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiDeferredEventQueue_1_1QueueNode.html) for details

---
