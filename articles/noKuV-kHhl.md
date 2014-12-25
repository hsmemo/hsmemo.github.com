---
layout: default
title: JvmtiEnvBase クラス関連のクラス (JvmtiEnvBase, JvmtiEnvIterator, VM_GetOwnedMonitorInfo, VM_GetObjectMonitorUsage, VM_GetCurrentContendedMonitor, VM_GetStackTrace, VM_GetMultipleStackTraces, VM_GetAllStackTraces, VM_GetThreadListStackTraces, VM_GetFrameCount, VM_GetFrameLocation, ResourceTracker, JvmtiMonitorClosure, 及びそれらの補助クラス(ThreadInsideIterationClosure))
---
[Top](../index.html)

#### JvmtiEnvBase クラス関連のクラス (JvmtiEnvBase, JvmtiEnvIterator, VM_GetOwnedMonitorInfo, VM_GetObjectMonitorUsage, VM_GetCurrentContendedMonitor, VM_GetStackTrace, VM_GetMultipleStackTraces, VM_GetAllStackTraces, VM_GetThreadListStackTraces, VM_GetFrameCount, VM_GetFrameLocation, ResourceTracker, JvmtiMonitorClosure, 及びそれらの補助クラス(ThreadInsideIterationClosure))

これらは, JVMTI の機能を実装するために使われているクラス.
より具体的に言うと, "JVMTI environment" (JVMTI 環境) を実現するためのクラス.


### クラス一覧(class list)

  * [JvmtiEnvBase](#noQh06Y--q)
  * [JvmtiEnvIterator](#noNr-5CRr-)
  * [VM_GetOwnedMonitorInfo](#normcspNjc)
  * [VM_GetObjectMonitorUsage](#nozlE40qRB)
  * [VM_GetCurrentContendedMonitor](#noTbtzqrZR)
  * [VM_GetStackTrace](#no9vXRIeOJ)
  * [VM_GetMultipleStackTraces](#noT0M38LVz)
  * [VM_GetAllStackTraces](#no6jX8fs2c)
  * [VM_GetThreadListStackTraces](#notYxt6uZJ)
  * [VM_GetFrameCount](#noztDYXDOZ)
  * [VM_GetFrameLocation](#noberZOvVg)
  * [ResourceTracker](#noHLAJdNju)
  * [JvmtiMonitorClosure](#nocmll6hy3)
  * [ThreadInsideIterationClosure](#nonmZdyPRw)


---
## <a name="noQh06Y--q" id="noQh06Y--q">JvmtiEnvBase</a>

### 概要(Summary)
JVMTI の "JVMTI environment" (JVMTI 環境) を実装するクラスの基底クラス.

1つの JvmtiEnvEnv オブジェクトが 1つの JVMTI environment (= JVMTI エージェントが GetEnv() で取得する jvmtiEnv オブジェクト) に対応する (See: [here](no3718uqQ.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // One JvmtiEnv object is created per jvmti attachment;
    // done via JNI GetEnv() call. Multiple attachments are
    // allowed in jvmti.
    
    class JvmtiEnvBase : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス (See: JvmtiEnv).

(なお, JVMTI environment には JVMTI connection (JVMTI 接続) 毎の状態を管理する役割と
JVMTI 関数へのポインタ(environment pointer)を提供する役割があるが,
このクラスは状態管理を行う機能と JVMTI 関数内で使われる補助関数を提供している.
ただし, 一部の状態は JvmtiEnvThreadState クラスや JvmtiThreadState クラスが管理している. (See: [here](no3718uqQ.html) for details))




### 詳細(Details)
See: [here](../doxygen/classJvmtiEnvBase.html) for details

---
## <a name="noNr-5CRr-" id="noNr-5CRr-">JvmtiEnvIterator</a>

### 概要(Summary)
JVMTI の機能を実装するために使われている補助クラス(StackObjクラス).

生成された全ての JvmtiEnv オブジェクトをたどるためのイテレータクラス.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // This class is the only safe means of iterating through environments.
    // Note that this iteratation includes invalid environments pending
    // deallocation -- in fact, some uses depend on this behavior.
    
    class JvmtiEnvIterator : public StackObj {
```

### 使われ方(Usage)
JVMTI 関連の様々な処理で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classJvmtiEnvIterator.html) for details

---
## <a name="normcspNjc" id="normcspNjc">VM_GetOwnedMonitorInfo</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラス(VM_Operationクラス).

指定されたスレッドが所有するモニターの情報(およびそれらのモニターをロックしているスタックフレームの深さ)を取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // VM operation to get monitor information with stack depth.
    class VM_GetOwnedMonitorInfo : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEnv::GetOwnedMonitorInfo() 内, 及び JvmtiEnv::GetOwnedMonitorStackDepthInfo() 内で(のみ)使用されている (See: [here](no2935QGn.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GetOwnedMonitorInfo.html) for details

---
## <a name="nozlE40qRB" id="nozlE40qRB">VM_GetObjectMonitorUsage</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラス(VM_Operationクラス).

指定されたオブジェクトのモニター情報を取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // VM operation to get object monitor usage.
    class VM_GetObjectMonitorUsage : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEnv::GetObjectMonitorUsage() 内で(のみ)使用されている (See: [here](no2935QNb.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GetObjectMonitorUsage.html) for details

---
## <a name="noTbtzqrZR" id="noTbtzqrZR">VM_GetCurrentContendedMonitor</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラス(VM_Operationクラス).

指定されたスレッドが現在モニターによってブロックされている場合
(java.lang.Object.wait() で待機しているか, モニター確保のためブロックしている場合), 
そのモニターの情報を取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // VM operation to get current contended monitor.
    class VM_GetCurrentContendedMonitor : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEnv::GetCurrentContendedMonitor() 内で(のみ)使用されている (See: [here](no2935DKJ.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GetCurrentContendedMonitor.html) for details

---
## <a name="no9vXRIeOJ" id="no9vXRIeOJ">VM_GetStackTrace</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラス(VM_Operationクラス).

指定されたスレッドのスタックトレースを取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // VM operation to get stack trace at safepoint.
    class VM_GetStackTrace : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEnv::GetStackTrace() 内で(のみ)使用されている (See: [here](no29353yh.html) for details)




### 詳細(Details)
See: [here](../doxygen/classVM__GetStackTrace.html) for details

---
## <a name="noT0M38LVz" id="noT0M38LVz">VM_GetMultipleStackTraces</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラスの基底クラス.

指定された複数のスレッドのスタックトレースを取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // VM operation to get stack trace at safepoint.
    class VM_GetMultipleStackTraces : public VM_Operation {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classVM__GetMultipleStackTraces.html) for details

---
## <a name="no6jX8fs2c" id="no6jX8fs2c">VM_GetAllStackTraces</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラス(VM_Operationクラス).
VM_GetMultipleStackTraces クラスの具象サブクラスの1つ.

全てのスレッドのスタックトレースを取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // VM operation to get stack trace at safepoint.
    class VM_GetAllStackTraces : public VM_GetMultipleStackTraces {
```

### 使われ方(Usage)
JvmtiEnv::GetAllStackTraces() 内で(のみ)使用されている (See: [here](no2935eR0.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GetAllStackTraces.html) for details

---
## <a name="notYxt6uZJ" id="notYxt6uZJ">VM_GetThreadListStackTraces</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラス(VM_Operationクラス).
VM_GetMultipleStackTraces クラスの具象サブクラスの1つ.

複数の指定されたスレッドのスタックトレースを取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // VM operation to get stack trace at safepoint.
    class VM_GetThreadListStackTraces : public VM_GetMultipleStackTraces {
```

### 使われ方(Usage)
JvmtiEnv::GetThreadListStackTraces() 内で(のみ)使用されている (See: [here](no2935eR0.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GetThreadListStackTraces.html) for details

---
## <a name="noztDYXDOZ" id="noztDYXDOZ">VM_GetFrameCount</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラス(VM_Operationクラス).

指定されたスレッドのスタック中にあるスタックフレームの数を取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // VM operation to count stack frames at safepoint.
    class VM_GetFrameCount : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEnv::GetFrameCount() 内で(のみ)使用されている (See: [here](no2935q2D.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GetFrameCount.html) for details

---
## <a name="noberZOvVg" id="noberZOvVg">VM_GetFrameLocation</a>

### 概要(Summary)
JvmtiEnv クラス内で使用される補助クラス(VM_Operationクラス).

指定されたスタックフレームについて, 現在の実行地点(実行しているメソッド, および bci(bytecode index))を取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // VM operation to frame location at safepoint.
    class VM_GetFrameLocation : public VM_Operation {
```

### 使われ方(Usage)
JvmtiEnv::GetFrameLocation() 内で(のみ)使用されている (See: [here](no2935RVW.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__GetFrameLocation.html) for details

---
## <a name="noHLAJdNju" id="noHLAJdNju">ResourceTracker</a>

### 概要(Summary)
JvmtiExtensions クラス内で使用される補助クラス (See: JvmtiExtensions).

「メモリ確保処理を複数回行い, そのうち一つでも失敗したら全ての確保処理をキャンセルする(確保した領域を全て開放する)」
という処理パターンを簡単に記述するための補助クラス(StackObjクラス).
このクラスの ResourceTracker::allocate() メソッド経由で確保を行うと, 
1つでも失敗した場合はデストラクタで全て開放してくれる.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // ResourceTracker
    //
    // ResourceTracker works a little like a ResourceMark. All allocates
    // using the resource tracker are recorded. If an allocate using the
    // resource tracker fails the destructor will free any resources
    // that were allocated using the tracker.
    // The motive for this class is to avoid messy error recovery code
    // in situations where multiple allocations are done in sequence. If
    // the second or subsequent allocation fails it avoids any code to
    // release memory allocated in the previous calls.
    //
    // Usage :-
    //   ResourceTracker rt(env);
    //   :
    //   err = rt.allocate(1024, &ptr);
    
    class ResourceTracker : public StackObj {
```

### 使われ方(Usage)
JvmtiExtensions::get_functions() 内, 及び JvmtiExtensions::get_events() 内で(のみ)使用されている (See: [here](no2935nLg.html) for details).




### 詳細(Details)
See: [here](../doxygen/classResourceTracker.html) for details

---
## <a name="nocmll6hy3" id="nocmll6hy3">JvmtiMonitorClosure</a>

### 概要(Summary)
JvmtiEnvBase クラス内で使用される補助クラス.

指定されたスレッドが所有するモニターの情報(およびそれらのモニターをロックしているスタックフレームの深さ)を取得する.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.hpp))
    // Jvmti monitor closure to collect off stack monitors.
    class JvmtiMonitorClosure: public MonitorClosure {
```

### 使われ方(Usage)
JvmtiEnvBase::get_owned_monitors() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classJvmtiMonitorClosure.html) for details

---
## <a name="nonmZdyPRw" id="nonmZdyPRw">ThreadInsideIterationClosure</a>

### 概要(Summary)
JvmtiEnvBase::check_for_periodic_clean_up() 内で定義されているローカルクラス.

JVMTI environment を辿る処理が現在行われているかどうかを調べる.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.cpp))
      class ThreadInsideIterationClosure: public ThreadClosure {
```

### 使われ方(Usage)
JvmtiEnvBase::check_for_periodic_clean_up() 内で(のみ)使用されている.
そして, この関数は JvmtiGCMarker::JvmtiGCMarker() 内で(のみ)呼び出されている (See: JvmtiGCMarker).




### 詳細(Details)
See: [here](../doxygen/classThreadInsideIterationClosure.html) for details

---
