---
layout: default
title: JvmtiExport クラス関連のクラス (JvmtiExport, JvmtiCodeBlobDesc, JvmtiEventCollector, JvmtiDynamicCodeEventCollector, JvmtiVMObjectAllocEventCollector, NoJvmtiVMObjectAllocMark, JvmtiGCMarker, JvmtiHideSingleStepping, 及びそれらの補助クラス(JvmtiJavaThreadEventTransition, JvmtiThreadEventTransition, JvmtiEventMark, JvmtiThreadEventMark, JvmtiClassEventMark, JvmtiMethodEventMark, JvmtiLocationEventMark, JvmtiExceptionEventMark, JvmtiClassFileLoadEventMark, JvmtiClassFileLoadHookPoster, JvmtiVMObjectAllocEventMark, JvmtiCompiledMethodLoadEventMark, JvmtiMonitorEventMark))
---
[Top](../index.html)

#### JvmtiExport クラス関連のクラス (JvmtiExport, JvmtiCodeBlobDesc, JvmtiEventCollector, JvmtiDynamicCodeEventCollector, JvmtiVMObjectAllocEventCollector, NoJvmtiVMObjectAllocMark, JvmtiGCMarker, JvmtiHideSingleStepping, 及びそれらの補助クラス(JvmtiJavaThreadEventTransition, JvmtiThreadEventTransition, JvmtiEventMark, JvmtiThreadEventMark, JvmtiClassEventMark, JvmtiMethodEventMark, JvmtiLocationEventMark, JvmtiExceptionEventMark, JvmtiClassFileLoadEventMark, JvmtiClassFileLoadHookPoster, JvmtiVMObjectAllocEventMark, JvmtiCompiledMethodLoadEventMark, JvmtiMonitorEventMark))

これらは, JVMTI の機能を実装するために使われているクラス.

(なお, このうちの多くは JVMTI のイベント通知機能を実装するためのクラス (See: [here](no29359PS.html) for details))


### クラス一覧(class list)

  * [JvmtiExport](#no5_Jb1FkJ)
  * [JvmtiCodeBlobDesc](#nomfeAY11N)
  * [JvmtiEventCollector](#no_ELxTRTB)
  * [JvmtiDynamicCodeEventCollector](#noscRZjOMS)
  * [JvmtiVMObjectAllocEventCollector](#noI_IOjutr)
  * [NoJvmtiVMObjectAllocMark](#noYQFzyG1v)
  * [JvmtiGCMarker](#no4rTC1c-S)
  * [JvmtiHideSingleStepping](#noL34L3_h7)
  * [JvmtiJavaThreadEventTransition](#no5mDm1mBW)
  * [JvmtiThreadEventTransition](#noso59cbpW)
  * [JvmtiEventMark](#noth5oEWX9)
  * [JvmtiThreadEventMark](#nowaW8xiUS)
  * [JvmtiClassEventMark](#nolyrPnHk8)
  * [JvmtiMethodEventMark](#noxIXH_j31)
  * [JvmtiLocationEventMark](#no9at4bjG9)
  * [JvmtiExceptionEventMark](#nof56u_J0-)
  * [JvmtiClassFileLoadEventMark](#noKuAyi4R8)
  * [JvmtiClassFileLoadHookPoster](#noKvnPenv-)
  * [JvmtiVMObjectAllocEventMark](#nocUZa-SGp)
  * [JvmtiCompiledMethodLoadEventMark](#noAbZr5qbw)
  * [JvmtiMonitorEventMark](#nogEUqeNbQ)


---
## <a name="no5_Jb1FkJ" id="no5_Jb1FkJ">JvmtiExport</a>

### 概要(Summary)
JVMTI 関係の関数を納めた名前空間(AllStatic クラス).

このクラスは, HotSpot の JVMTI 関係の部分を HotSpot のそれ以外の部分から隠蔽する役割を果たしている
(HotSpot 内の他のコードからは, このクラスを介して JVMTI の機能にアクセスする)
(といっても, その他の部分から JVMTI に働きかけるのは主にイベント通知時ぐらいだが...)
(See: [here](no3718uqQ.html) and [here](no2935C7Z.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
    // This class contains the JVMTI interface for the rest of hotspot.
    //
    class JvmtiExport : public AllStatic {
```

### 使われ方(Usage)
JVMTI 関連の様々な処理で使用されている (#TODO).

### 内部構造(Internal structure)
内部には, JVMTI 処理のための以下のようなメソッドが定義されている.

(例: JVMTI のバージョンを確認するメソッド, JVMTI environment を取得するためのメソッド)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
      static bool is_jvmti_version(jint version)                      { return (version & JVMTI_VERSION_MASK) == JVMTI_VERSION_VALUE; }
      static bool is_jvmdi_version(jint version)                      { return (version & JVMTI_VERSION_MASK) == JVMDI_VERSION_VALUE; }
      static jint get_jvmti_interface(JavaVM *jvm, void **penv, jint version);
```

(例: JVMTI agent が各種の capability を取得しているかどうかを設定／取得するメソッド)
(なお, JvmtiExport::set_can_*() メソッド及び JvmtiExport::can_*() メソッドは, 
以下のように JVMTI_SUPPORT_FLAG というマクロを使って間接的に定義されている(後述))


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
      JVMTI_SUPPORT_FLAG(can_get_source_debug_extension)
      JVMTI_SUPPORT_FLAG(can_maintain_original_method_order)
      JVMTI_SUPPORT_FLAG(can_post_interpreter_events)
      JVMTI_SUPPORT_FLAG(can_post_on_exceptions)
      JVMTI_SUPPORT_FLAG(can_post_breakpoint)
      JVMTI_SUPPORT_FLAG(can_post_field_access)
      JVMTI_SUPPORT_FLAG(can_post_field_modification)
      JVMTI_SUPPORT_FLAG(can_post_method_entry)
      JVMTI_SUPPORT_FLAG(can_post_method_exit)
      JVMTI_SUPPORT_FLAG(can_pop_frame)
      JVMTI_SUPPORT_FLAG(can_force_early_return)
```

(例: JVMTI の各イベントを通知する必要があるかどうかを設定／取得するメソッド)
(なお, JvmtiExport::set_should_post_*() メソッド及び JvmtiExport::should_post_*() メソッドは, 
以下のように JVMTI_SUPPORT_FLAG というマクロを使って間接的に定義されている(後述))


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
      JVMTI_SUPPORT_FLAG(should_post_single_step)
      JVMTI_SUPPORT_FLAG(should_post_field_access)
      JVMTI_SUPPORT_FLAG(should_post_field_modification)
      JVMTI_SUPPORT_FLAG(should_post_class_load)
      JVMTI_SUPPORT_FLAG(should_post_class_prepare)
      JVMTI_SUPPORT_FLAG(should_post_class_unload)
      JVMTI_SUPPORT_FLAG(should_post_native_method_bind)
      JVMTI_SUPPORT_FLAG(should_post_compiled_method_load)
      JVMTI_SUPPORT_FLAG(should_post_compiled_method_unload)
      JVMTI_SUPPORT_FLAG(should_post_dynamic_code_generated)
      JVMTI_SUPPORT_FLAG(should_post_monitor_contended_enter)
      JVMTI_SUPPORT_FLAG(should_post_monitor_contended_entered)
      JVMTI_SUPPORT_FLAG(should_post_monitor_wait)
      JVMTI_SUPPORT_FLAG(should_post_monitor_waited)
      JVMTI_SUPPORT_FLAG(should_post_data_dump)
      JVMTI_SUPPORT_FLAG(should_post_garbage_collection_start)
      JVMTI_SUPPORT_FLAG(should_post_garbage_collection_finish)
      JVMTI_SUPPORT_FLAG(should_post_on_exceptions)
    
      // ------ the below maybe don't have to be (but are for now)
      // fixed conditions here ------------
      // any events can be enabled
      JVMTI_SUPPORT_FLAG(should_post_thread_life)
      JVMTI_SUPPORT_FLAG(should_post_object_free)
      JVMTI_SUPPORT_FLAG(should_post_resource_exhausted)
    
      // we are holding objects on the heap - need to talk to GC - e.g.
      // breakpoint info
      JVMTI_SUPPORT_FLAG(should_clean_up_heap_objects)
      JVMTI_SUPPORT_FLAG(should_post_vm_object_alloc)
```

(例: JVMTI の各種イベントの通知を行うメソッド (JvmtiExport::post_*() メソッド))
(JvmtiExport::should_*() で調べて JvmtiExport::post_*(), というパターンが基本)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
      static void post_field_modification(JavaThread *thread, methodOop method, address location,
                                          KlassHandle field_klass, Handle object, jfieldID field,
                                          char sig_type, jvalue *value);
    
    
      // posts a DynamicCodeGenerated event (internal/private implementation).
      // The public post_dynamic_code_generated* functions make use of the
      // internal implementation.  Also called from JvmtiDeferredEvent::post()
      static void post_dynamic_code_generated_internal(const char *name, const void *code_begin, const void *code_end) KERNEL_RETURN;
```


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
      // Methods that notify the debugger that something interesting has happened in the VM.
      static void post_vm_start              ();
      static void post_vm_initialized        ();
      static void post_vm_death              ();
    
      static void post_single_step           (JavaThread *thread, methodOop method, address location) KERNEL_RETURN;
      static void post_raw_breakpoint        (JavaThread *thread, methodOop method, address location) KERNEL_RETURN;
    
      static void post_exception_throw       (JavaThread *thread, methodOop method, address location, oop exception) KERNEL_RETURN;
      static void notice_unwind_due_to_exception (JavaThread *thread, methodOop method, address location, oop exception, bool in_handler_frame) KERNEL_RETURN;
```


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
      static void post_method_entry          (JavaThread *thread, methodOop method, frame current_frame) KERNEL_RETURN;
      static void post_method_exit           (JavaThread *thread, methodOop method, frame current_frame) KERNEL_RETURN;
    
      static void post_class_load            (JavaThread *thread, klassOop klass) KERNEL_RETURN;
      static void post_class_unload          (klassOop klass) KERNEL_RETURN;
      static void post_class_prepare         (JavaThread *thread, klassOop klass) KERNEL_RETURN;
    
      static void post_thread_start          (JavaThread *thread) KERNEL_RETURN;
      static void post_thread_end            (JavaThread *thread) KERNEL_RETURN;
```

### 備考(Notes)
JVMTI_SUPPORT_FLAG マクロは以下のように定義されている
(フィールド, 及びそのフィールドに対するアクセサメソッドをまとめて定義する).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
    #ifndef JVMTI_KERNEL
    #define JVMTI_SUPPORT_FLAG(key)                                         \
      private:                                                              \
      static bool  _##key;                                                  \
      public:                                                               \
      inline static void set_##key(bool on)       { _##key = (on != 0); }   \
      inline static bool key()                    { return _##key; }
    #else  // JVMTI_KERNEL
    #define JVMTI_SUPPORT_FLAG(key)                                           \
      private:                                                                \
      const static bool _##key = false;                                       \
      public:                                                                 \
      inline static void set_##key(bool on)       { report_unsupported(on); } \
      inline static bool key()                    { return _##key; }
    #endif // JVMTI_KERNEL
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiExport.html) for details

---
## <a name="nomfeAY11N" id="nomfeAY11N">JvmtiCodeBlobDesc</a>

### 概要(Summary)
JVMTI 関係の処理で使用されているユーティリティ・クラス.

処理対象の CodeBlob の情報(名前およびコードのアドレス)を格納しておくためのクラス (See: [here](no29359Bq.html) and [here](no2935lCe.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
    // Support class used by JvmtiDynamicCodeEventCollector and others. It
    // describes a single code blob by name and address range.
    class JvmtiCodeBlobDesc : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 JvmtiDynamicCodeEventCollector オブジェクトの _code_blobs フィールド
* 各 CodeBlobCollector オブジェクトの _code_blobs フィールド
* CodeBlobCollector クラスの _global_code_blobs フィールド (static フィールド)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* JvmtiDynamicCodeEventCollector::register_stub()

  (See: [here](no29359Bq.html) for details)

* CodeBlobCollector::do_blob()

  (See: [here](no2935lCe.html) for details)

* CodeBlobCollector::collect()

  (See: [here](no2935lCe.html) for details)

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む
(そして, メソッドはこれらのフィールドへの getter メソッド(アクセサメソッド)のみ)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
      char _name[64];
      address _code_begin;
      address _code_end;
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiCodeBlobDesc.html) for details

---
## <a name="no_ELxTRTB" id="no_ELxTRTB">JvmtiEventCollector</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するためのクラス (の基底クラス).

より具体的に言うと, 「ソースコード中のあるスコープの間だけはイベントの通知処理をすぐに行わずに溜めておき, 
後でまとめて行いたい」という場面で使用される補助クラス(StackObjクラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
    // JvmtiEventCollector is a helper class to setup thread for
    // event collection.
    class JvmtiEventCollector : public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiEventCollector.html) for details

---
## <a name="noscRZjOMS" id="noscRZjOMS">JvmtiDynamicCodeEventCollector</a>

### 概要(Summary)
JvmtiEventCollector クラスの具象サブクラスの1つ.
このクラスは DynamicCodeGenerated イベントの通知処理に使用される (See: [here](no29359Bq.html) for details).

このクラスは JvmtiExport::post_dynamic_code_generated_while_holding_locks() メソッドと合わせて使用する
(通常時には DynamicCodeGenerated イベントの通知は JvmtiExport::post_dynamic_code_generated() メソッドで行うが, 
このメソッドはロックを持っている間は呼び出せない.
一方, JvmtiExport::post_dynamic_code_generated_while_holding_locks() メソッドは, 
ロックを持っている間でも呼び出すことが出来る.
といってもロック保持中に通知するわけではなく, イベントは一時的に JvmtiDynamicCodeEventCollector オブジェクト内に溜められる.
そして, その JvmtiDynamicCodeEventCollector オブジェクトが(スコープを抜けて)死ぬときに, 
溜めていたイベントが JvmtiExport::post_dynamic_code_generated() で通知される.)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
    // A JvmtiDynamicCodeEventCollector is a helper class for the JvmtiExport
    // interface. It collects "dynamic code generated" events that are posted
    // while holding locks. When the event collector goes out of scope the
    // events will be posted.
    //
    // Usage :-
    //
    // {
    //   JvmtiDynamicCodeEventCollector event_collector;
    //   :
    //   { MutexLocker ml(...)
    //     :
    //     JvmtiExport::post_dynamic_code_generated_while_holding_locks(...)
    //   }
    //   // event collector goes out of scope => post events to profiler.
    // }
    
    class JvmtiDynamicCodeEventCollector : public JvmtiEventCollector {
```


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
      // similiar to post_dynamic_code_generated except that it can be used to
      // post a DynamicCodeGenerated event while holding locks in the VM. Any event
      // posted using this function is recorded by the enclosing event collector
      // -- JvmtiDynamicCodeEventCollector.
      static void post_dynamic_code_generated_while_holding_locks(const char* name, address code_begin, address code_end) KERNEL_RETURN;
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で JvmtiDynamicCodeEventCollector 型の局所変数を宣言するだけ.

#### インスタンスの格納場所(where its instances are stored)
各 JvmtiThreadState オブジェクトの _dynamic_code_event_collector フィールドに(のみ)格納されている.

(正確には, このフィールドは JvmtiDynamicCodeEventCollector の線形リストを格納するフィールド.
JvmtiDynamicCodeEventCollector オブジェクトは _prev フィールドで次の
JvmtiDynamicCodeEventCollector オブジェクトを指せる構造になっている.
その JvmtiThreadState オブジェクト内で生成した JvmtiDynamicCodeEventCollector オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 使用箇所(where its instances are used)
SharedRuntime::handle_ic_miss_helper() 内で(のみ)使用されている

(正確には SharedRuntime::handle_ic_miss_helper() の中で JvmtiDynamicCodeEventCollector 型の局所変数が宣言され,
その後ここから呼び出される処理の中でイベント通知情報が蓄えられる).




### 詳細(Details)
See: [here](../doxygen/classJvmtiDynamicCodeEventCollector.html) for details

---
## <a name="noI_IOjutr" id="noI_IOjutr">JvmtiVMObjectAllocEventCollector</a>

### 概要(Summary)
JvmtiEventCollector クラスの具象サブクラスの1つ.
このクラスは VMObjectAlloc イベントの通知処理に使用される (See: [here](no2935wwv.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
    // Used to record vm internally allocated object oops and post
    // vm object alloc event for objects visible to java world.
    // Constructor enables JvmtiThreadState flag and all vm allocated
    // objects are recorded in a growable array. When destructor is
    // called the vm object alloc event is posted for each objects
    // visible to java world.
    // See jvm.cpp file for its usage.
    //
    class JvmtiVMObjectAllocEventCollector : public JvmtiEventCollector {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で JvmtiVMObjectAllocEventCollector 型の局所変数を宣言するだけ.

#### インスタンスの格納場所(where its instances are stored)
各 JvmtiThreadState オブジェクトの _vm_object_alloc_event_collector フィールドに(のみ)格納されている.

(正確には, このフィールドは JvmtiVMObjectAllocEventCollector の線形リストを格納するフィールド.
JvmtiVMObjectAllocEventCollector オブジェクトは _prev フィールドで次の
JvmtiVMObjectAllocEventCollector オブジェクトを指せる構造になっている.
その JvmtiThreadState オブジェクト内で生成した JvmtiVMObjectAllocEventCollector オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 使用箇所(where its instances are used)
メモリを確保する可能性がある各種の CVMI 関数内で(のみ)使用されている

(正確にはそれらの関数中で JvmtiVMObjectAllocEventCollector 型の局所変数が宣言され,
その後そこから呼び出される処理の中でイベント通知情報が蓄えられる).




### 詳細(Details)
See: [here](../doxygen/classJvmtiVMObjectAllocEventCollector.html) for details

---
## <a name="noYQFzyG1v" id="noYQFzyG1v">NoJvmtiVMObjectAllocMark</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

(ソースコード中のあるスコープの間だけ, VMObjectAlloc イベントの通知処理を無効にするクラスのようだが... #TODO)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
    // Marker class to disable the posting of VMObjectAlloc events
    // within its scope.
    //
    // Usage :-
    //
    // {
    //   NoJvmtiVMObjectAllocMark njm;
    //   :
    //   // VMObjAlloc event will not be posted
    //   JvmtiExport::vm_object_alloc_event_collector(obj);
    //   :
    // }
    
    class NoJvmtiVMObjectAllocMark : public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classNoJvmtiVMObjectAllocMark.html) for details

---
## <a name="no4rTC1c-S" id="no4rTC1c-S">JvmtiGCMarker</a>

### 概要(Summary)
保守運用機能のためのクラス (JVMTI 用のフック点を管理するためのクラス).

JVMTI のフック点の生成処理を簡単に行うための補助クラス(StackObjクラス).
ソースコード上のスコープに連動して自動的にフック点を生成する (See: [here](no2935WqL.html) and [here](no2935j0R.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
    // Base class for reporting GC events to JVMTI.
    class JvmtiGCMarker : public StackObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 SvcGCMarker オブジェクトの _jgcm フィールドに(のみ)格納されている (See: SvcGCMarker).

#### 生成箇所(where its instances are created)
(SvcGCMarker クラスの _jgcm フィールドは, ポインタ型ではなく実体なので,
 SvcGCMarker オブジェクトの生成時に一緒に生成される)

### 内部構造(Internal structure)
コンストラクタで JvmtiExport::post_garbage_collection_start() を呼び出し, 
デストラクタで JvmtiExport::post_garbage_collection_finish() を呼び出す
(これらは JVMTI のフック点).

(なお, コンストラクタで JvmtiEnvBase::check_for_periodic_clean_up() を呼び出し, 
不要になった JvmtiEnvBase や JvmtiEnvThreadState を削除する役割も担っている.)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    JvmtiGCMarker::JvmtiGCMarker() {
      // if there aren't any JVMTI environments then nothing to do
      if (!JvmtiEnv::environments_might_exist()) {
        return;
      }
    
      if (JvmtiExport::should_post_garbage_collection_start()) {
        JvmtiExport::post_garbage_collection_start();
      }
    
      if (SafepointSynchronize::is_at_safepoint()) {
        // Do clean up tasks that need to be done at a safepoint
        JvmtiEnvBase::check_for_periodic_clean_up();
      }
    }
    
    JvmtiGCMarker::~JvmtiGCMarker() {
      // if there aren't any JVMTI environments then nothing to do
      if (!JvmtiEnv::environments_might_exist()) {
        return;
      }
    
      // JVMTI notify gc finish
      if (JvmtiExport::should_post_garbage_collection_finish()) {
        JvmtiExport::post_garbage_collection_finish();
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiGCMarker.html) for details

---
## <a name="noL34L3_h7" id="noL34L3_h7">JvmtiHideSingleStepping</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するためのクラス.
より具体的に言うと, SingleStep イベントの処理内で使用される補助クラス (See: [here](no7882EDP.html) for details).

ソースコード中のあるスコープの間だけは SingleStep イベントの通知を無効にしておきたい, 
という場面で使用される補助クラス(StackObjクラス).
(主に, ダイナミックロードが走る可能性がある場合で,
そのままだとクラスローダの処理まで SingleStep 実行されてしまうので一時的に SingleStep を無効にする, 
という用途で使われている模様)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
    // JvmtiHideSingleStepping is a helper class for hiding
    // internal single step events.
    class JvmtiHideSingleStepping : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* InterpreterRuntime::resolve_invoke()
* InterpreterRuntime::resolve_get_put()
* InterpreterRuntime::resolve_invokedynamic()

### 内部構造(Internal structure)
コンストラクタで JvmtiExport::hide_single_stepping() を呼んで SingleStep イベントを無効にし, 
デストラクタで JvmtiExport::expose_single_stepping() を呼んで元に戻している.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.hpp))
      JvmtiHideSingleStepping(JavaThread * thread) {
        assert(thread != NULL, "sanity check");
    
        _single_step_hidden = false;
        _thread = thread;
        if (JvmtiExport::should_post_single_step()) {
          _single_step_hidden = JvmtiExport::hide_single_stepping(_thread);
        }
      }
    
      ~JvmtiHideSingleStepping() {
        if (_single_step_hidden) {
          JvmtiExport::expose_single_stepping(_thread);
        }
      }
```

### 備考(Notes)
JvmtiHideSingleStepping の効果は累積する
(JvmtiHideSingleStepping が複数使用された場合,
全ての JvmtiHideSingleStepping のデストラクタが呼ばれるまで SingleStep イベントは有効状態に戻らない).

#### 参考(for your information): JvmtiExport::hide_single_stepping()
See: [here](no2935wFM.html) for details
#### 参考(for your information): JvmtiThreadState::set_hide_single_stepping()
See: [here](no2935j7F.html) for details
#### 参考(for your information): JvmtiExport::expose_single_stepping()
See: [here](no2935knw.html) for details
#### 参考(for your information): JvmtiThreadState::clear_hide_single_stepping()
See: [here](no2935xx2.html) for details



### 詳細(Details)
See: [here](../doxygen/classJvmtiHideSingleStepping.html) for details

---
## <a name="no5mDm1mBW" id="no5mDm1mBW">JvmtiJavaThreadEventTransition</a>

### 概要(Summary)
保守運用機能のためのクラス (JVMTI 機能及び Dynamic Attach 機能用のクラス).
より具体的に言うと, JVMTI のイベント通知処理(コールバック呼び出し処理) や 
Dynamic Attach の Agent_OnAttach() 関数の呼び出し処理の記述を簡単に行うための補助クラス(StackObjクラス).

具体的には, 以下の機能を提供する.

* 呼び出し中だけ, JavaThreadState を _thread_in_native に変更する

  (コールバック関数や Agent_OnAttach() はネイティブ関数なので, 
   呼び出し中は _thread_in_native にしておかないと Safepoint が始まらない).

* 呼び出し処理で確保された ResourceObj を自動的に開放する

* 呼び出し処理で確保された Handle を自動的に開放する

なお, このクラスは _thread_in_vm 状態にある JavaThread 用. 
それ以外の場合用には JvmtiThreadEventTransition が用意されている (See: JvmtiThreadEventTransition).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    ///////////////////////////////////////////////////////////////
    //
    // JvmtiEventTransition
    //
    // TO DO --
    //  more handle purging
    
    // Use this for JavaThreads and state is  _thread_in_vm.
    class JvmtiJavaThreadEventTransition : StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no29359PS.html) and [here](no3026gMG.html) for details).

* JvmtiExport::post_*()
  (ただし, 一部のメソッドでは JvmtiThreadEventTransition が使用されている (See: JvmtiThreadEventTransition))
* JvmtiExport::notice_unwind_due_to_exception()
* JvmtiExport::load_agent_library()

### 内部構造(Internal structure)
実体としては, 単なる ResourceMark & ThreadToNativeFromVM & HandleMark のラッパー
(See: ResourceMark, ThreadToNativeFromVM, HandleMark).




### 詳細(Details)
See: [here](../doxygen/classJvmtiJavaThreadEventTransition.html) for details

---
## <a name="noso59cbpW" id="noso59cbpW">JvmtiThreadEventTransition</a>

### 概要(Summary)
特殊な JvmtiJavaThreadEventTransition クラス (See: JvmtiJavaThreadEventTransition).

このクラスは, _thread_in_vm 状態ではない JavaThread や JavaThread 以外のスレッド用.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    // For JavaThreads which are not in _thread_in_vm state
    // and other system threads use this.
    class JvmtiThreadEventTransition : StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no29359PS.html) for details).

* JvmtiExport::post_garbage_collection_finish()
* JvmtiExport::post_garbage_collection_start()
* JvmtiExport::post_data_dump()
* JvmtiExport::post_monitor_contended_enter()
* JvmtiExport::post_monitor_contended_entered()
* JvmtiExport::post_monitor_wait()
* JvmtiExport::post_monitor_waited()




### 詳細(Details)
See: [here](../doxygen/classJvmtiThreadEventTransition.html) for details

---
## <a name="noth5oEWX9" id="noth5oEWX9">JvmtiEventMark</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するためのクラス.
より具体的に言うと, 実際のイベント通知処理(コールバック呼び出し処理)の記述を簡単に行うための補助クラス(StackObjクラス).

具体的には, 以下のような機能を提供する.

* コールバックに渡す引数を JNI handle 化する
  (コールバックはネイティブ関数なので, 呼び出し中は JNI Handle 化しておかないと保護されない).
* そのために使用する JNIHandleBlock を自動的に確保/開放する.
* コールバック呼び出しの前後で, JvmtiThreadState の例外に関する状態 (is_exception_detected(), is_exception_caught()) 
  を自動的に待避/復帰する.
* スタックフレームを辿れる状態にする.

```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    ///////////////////////////////////////////////////////////////
    //
    // JvmtiEventMark
    //
    
    class JvmtiEventMark : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no29359PS.html) for details).

* JvmtiExport::post_vm_death()
* JvmtiExport::post_compiled_method_unload()
* JvmtiExport::post_compiled_method_load()
* JvmtiExport::post_dynamic_code_generated_internal()
* JvmtiExport::post_dynamic_code_generated()

### 内部構造(Internal structure)
コンストラクタで JNIHandleBlock の確保と JvmtiThreadState の状態の待避を行い,
デストラクタで JNIHandleBlock や JvmtiThreadState の状態を元に戻している.

また, 引数を JNI Handle 化するための補助関数も提供している (この補助関数はサブクラス内で使用されている).

(なお, 本当は #if 0 の中のように書きたかったけど今のところ上手く動かないのでとりあえずの実装にしている, とのこと)

```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    #if 0
      jobject to_jobject(oop obj) { return obj == NULL? NULL : _hblock->allocate_handle_fast(obj); }
    #else
      // we want to use the code above - but that needs the JNIHandle changes - later...
      // for now, use regular make_local
      jobject to_jobject(oop obj) { return JNIHandles::make_local(_thread,obj); }
    #endif
```

#### 参考(for your information): JvmtiEventMark::JvmtiEventMark()
See: [here](no2935Xke.html) for details
#### 参考(for your information): JvmtiEventMark::~JvmtiEventMark()
See: [here](no2935kuk.html) for details



### 詳細(Details)
See: [here](../doxygen/classJvmtiEventMark.html) for details

---
## <a name="nowaW8xiUS" id="nowaW8xiUS">JvmtiThreadEventMark</a>

### 概要(Summary)
JvmtiEventMark クラスのサブクラス.

JvmtiEventMark が JNI Handle 化して保護する範囲に加えて, jthread 型の値を1つ保護対象に加えている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiThreadEventMark : public JvmtiEventMark {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no29359PS.html) and [here](no3026gMG.html) for details).

* JvmtiExport::post_vm_start()
* JvmtiExport::post_vm_initialized()
* JvmtiExport::post_thread_start()
* JvmtiExport::post_thread_end()
* JvmtiExport::post_resource_exhausted()
* JvmtiExport::load_agent_library()




### 詳細(Details)
See: [here](../doxygen/classJvmtiThreadEventMark.html) for details

---
## <a name="nolyrPnHk8" id="nolyrPnHk8">JvmtiClassEventMark</a>

### 概要(Summary)
JvmtiThreadEventMark クラスのサブクラス.

JvmtiThreadEventMark が JNI Handle 化して保護する範囲に加えて, jclass 型の値を1つ保護対象に加えている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiClassEventMark : public JvmtiThreadEventMark {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no29359PS.html) for details).

* JvmtiExport::post_class_load()
* JvmtiExport::post_class_prepare()




### 詳細(Details)
See: [here](../doxygen/classJvmtiClassEventMark.html) for details

---
## <a name="noxIXH_j31" id="noxIXH_j31">JvmtiMethodEventMark</a>

### 概要(Summary)
JvmtiThreadEventMark クラスのサブクラス.

JvmtiThreadEventMark が JNI Handle 化して保護する範囲に加えて, jmethodID 型の値を1つ保護対象に加えている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiMethodEventMark : public JvmtiThreadEventMark {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no29359PS.html) for details).

* JvmtiExport::post_method_entry()
* JvmtiExport::post_method_exit()
* JvmtiExport::post_native_method_bind()




### 詳細(Details)
See: [here](../doxygen/classJvmtiMethodEventMark.html) for details

---
## <a name="no9at4bjG9" id="no9at4bjG9">JvmtiLocationEventMark</a>

### 概要(Summary)
JvmtiMethodEventMark クラスのサブクラス.

JvmtiMethodEventMark が JNI Handle 化して保護する範囲に加えて, jlocation 型の値を1つ保護対象に加えている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiLocationEventMark : public JvmtiMethodEventMark {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no29359PS.html) for details).

* JvmtiExport::post_raw_breakpoint()
* JvmtiExport::post_single_step()
* JvmtiExport::post_field_access()
* JvmtiExport::post_field_modification()




### 詳細(Details)
See: [here](../doxygen/classJvmtiLocationEventMark.html) for details

---
## <a name="nof56u_J0-" id="nof56u_J0-">JvmtiExceptionEventMark</a>

### 概要(Summary)
JvmtiLocationEventMark クラスのサブクラス.

JvmtiLocationEventMark が JNI Handle 化して保護する範囲に加えて, jobject 型の値(例外オブジェクト)を1つ保護対象に加えている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiExceptionEventMark : public JvmtiLocationEventMark {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no29359PS.html) for details).

* JvmtiExport::post_exception_throw()
* JvmtiExport::notice_unwind_due_to_exception()




### 詳細(Details)
See: [here](../doxygen/classJvmtiExceptionEventMark.html) for details

---
## <a name="noKuAyi4R8" id="noKuAyi4R8">JvmtiClassFileLoadEventMark</a>

### 概要(Summary)
JvmtiThreadEventMark クラスのサブクラス.

JvmtiThreadEventMark が JNI Handle 化して保護する範囲に加えて, 
クラスファイルのロードに関連するいくつかの値を保護対象に加えている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiClassFileLoadEventMark : public JvmtiThreadEventMark {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JvmtiClassFileLoadHookPoster::post_to_env()

そして, この関数は現在は以下のパスで(のみ)呼び出されている (See: [here](no2935WjX.html) for details).

```
JvmtiExport::post_class_file_load_hook()
-> JvmtiClassFileLoadHookPoster::post()
   -> JvmtiClassFileLoadHookPoster::post_all_envs()
      -> JvmtiClassFileLoadHookPoster::post_to_env()
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiClassFileLoadEventMark.html) for details

---
## <a name="noKvnPenv-" id="noKvnPenv-">JvmtiClassFileLoadHookPoster</a>

### 概要(Summary)
JVMTI のイベント通知機能を実装するための補助クラス(StackObjクラス).
より具体的に言うと, ClassFileLoadHook イベントの通知処理を行うためのクラス
(ClassFileLoadHook イベントの通知処理は, このクラスの JvmtiClassFileLoadHookPoster::post() メソッドに実装されている)
(See: [here](no2935WjX.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiClassFileLoadHookPoster : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no2935WjX.html) for details).

* JvmtiExport::post_class_file_load_hook()




### 詳細(Details)
See: [here](../doxygen/classJvmtiClassFileLoadHookPoster.html) for details

---
## <a name="nocUZa-SGp" id="nocUZa-SGp">JvmtiVMObjectAllocEventMark</a>

### 概要(Summary)
JvmtiClassEventMark クラスのサブクラス.

JvmtiClassEventMark が JNI Handle 化して保護する範囲に加えて, jobject 型の値(確保したオブジェクト)を1つ保護対象に加えている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiVMObjectAllocEventMark : public JvmtiClassEventMark  {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no2935wwv.html) for details).

* JvmtiExport::post_vm_object_alloc()




### 詳細(Details)
See: [here](../doxygen/classJvmtiVMObjectAllocEventMark.html) for details

---
## <a name="noAbZr5qbw" id="noAbZr5qbw">JvmtiCompiledMethodLoadEventMark</a>

### 概要(Summary)
JvmtiMethodEventMark クラスのサブクラス.

JvmtiMethodEventMark が JNI Handle 化して保護する範囲に加えて, 
CompiledMethodLoad イベントに関する幾つかの情報を保持している.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiCompiledMethodLoadEventMark : public JvmtiMethodEventMark {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no2935lCe.html) for details).

* JvmtiExport::post_compiled_method_load()




### 詳細(Details)
See: [here](../doxygen/classJvmtiCompiledMethodLoadEventMark.html) for details

---
## <a name="nogEUqeNbQ" id="nogEUqeNbQ">JvmtiMonitorEventMark</a>

### 概要(Summary)
JvmtiThreadEventMark クラスのサブクラス.

JvmtiThreadEventMark が JNI Handle 化して保護する範囲に加えて, jobject 型の値(ロック対象のオブジェクト)を1つ保護対象に加えている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiExport.cpp))
    class JvmtiMonitorEventMark : public JvmtiThreadEventMark {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている (See: [here](no29359PS.html) for details).

* JvmtiExport::post_monitor_contended_enter()
* JvmtiExport::post_monitor_contended_entered()
* JvmtiExport::post_monitor_wait()
* JvmtiExport::post_monitor_waited()




### 詳細(Details)
See: [here](../doxygen/classJvmtiMonitorEventMark.html) for details

---
