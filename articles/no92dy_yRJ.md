---
layout: default
title: JvmtiTrace クラス 
---
[Top](../index.html)

#### JvmtiTrace クラス 



---
## <a name="noaeyKLjiC" id="noaeyKLjiC">JvmtiTrace</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef JVMTI_TRACE 時にしか定義されない).

JVMTI に関するトレース出力用の関数や定数を納めた名前空間(AllStatic クラス).

```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiTrace.cpp))
    // class JvmtiTrace
    //
    // Support for JVMTI tracing code
    //
    // ------------
    // Usage:
    //    -XX:TraceJVMTI=DESC,DESC,DESC
    //
    //    DESC is   DOMAIN ACTION KIND
    //
    //    DOMAIN is function name
    //              event name
    //              "all" (all functions and events)
    //              "func" (all functions except boring)
    //              "allfunc" (all functions)
    //              "event" (all events)
    //              "ec" (event controller)
    //
    //    ACTION is "+" (add)
    //              "-" (remove)
    //
    //    KIND is
    //     for func
    //              "i" (input params)
    //              "e" (error returns)
    //              "o" (output)
    //     for event
    //              "t" (event triggered aka posted)
    //              "s" (event sent)
    //
    // Example:
    //            -XX:TraceJVMTI=ec+,GetCallerFrame+ie,Breakpoint+s
```


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiTrace.hpp))
    #ifdef JVMTI_TRACE
    
    class JvmtiTrace : AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiTrace.html) for details

---
