---
layout: default
title: DTraceJSDT クラス関連のクラス (DTraceJSDT, RegisteredProbes)
---
[Top](../index.html)

#### DTraceJSDT クラス関連のクラス (DTraceJSDT, RegisteredProbes)

これらは, 保守運用機能のためのクラス (DTrace JSDT 機能用のクラス) (See: [here](nof8Spkk-A.html) for details).


### クラス一覧(class list)

  * [DTraceJSDT](#noq4Dt3RZh)
  * [RegisteredProbes](#noWxfFrZxJ)


---
## <a name="noq4Dt3RZh" id="noq4Dt3RZh">DTraceJSDT</a>

### 概要(Summary)
保守運用機能のためのクラス (DTrace JSDT 機能用のクラス)
(別の言い方をすると sun.tracing.dtrace.JVM クラスを実現するためのクラス) (See: [here](nof8Spkk-A.html) for details).

DTrace JSDT 用の機能を納めた名前空間(AllStatic クラス).
DTrace JSDT 関係の機能は全てここに納められている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/dtraceJSDT.hpp))
    class DTraceJSDT : AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* sun.tracing.dtrace.JVM クラスの処理
  
  sun.tracing.dtrace.JVM.activate0()
  -> JVM_DTraceActivate()
     -> DTraceJSDT::activate()

  sun.tracing.dtrace.JVM.dispose0()
  -> JVM_DTraceDispose()
     -> DTraceJSDT::dispose()

  sun.tracing.dtrace.JVM.isEnabled0()
  -> JVM_DTraceIsProbeEnabled()
     -> DTraceJSDT::is_probe_enabled()

  sun.tracing.dtrace.JVM.isSupported0()
  -> JVM_DTraceIsSupported()
     -> DTraceJSDT::is_supported()
```




### 詳細(Details)
See: [here](../doxygen/classDTraceJSDT.html) for details

---
## <a name="noWxfFrZxJ" id="noWxfFrZxJ">RegisteredProbes</a>

### 概要(Summary)
DTraceJSDT クラス用の補助クラス.

DTraceJSDT::activate() で登録された内容を管理するためのクラス.
1つの RegisteredProbes オブジェクトが 1回の DTraceJSDT::activate() 呼び出しに対応する.

(より正確に言うと,
 DTraceJSDT::activate() はその内容を記録した RegisteredProbes オブジェクトを作成し, 
 そのポインタを jlong 値にキャストして返値とする.
 このため, DTraceJSDT::activate() の返値は RegisteredProbes オブジェクトそのもの)


```cpp
    ((cite: hotspot/src/share/vm/runtime/dtraceJSDT.hpp))
    class RegisteredProbes : public CHeapObj {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
DTraceJSDT::activate() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classRegisteredProbes.html) for details

---
