---
layout: default
title: Serviceability 機能 ： DTrace JSDT
---
[Up](noOQc_VTg2.html) [Top](../index.html)

#### Serviceability 機能 ： DTrace JSDT

--- 
## 概要(Summary)
内部的には DTraceJSDT クラスによって実現されている
(なお, 現状では Solaris 版の実装しかない (See: hotspot/src/os/solaris/vm/dtraceJSDT_solaris.cpp)).

DTraceJSDT::pd_activate() と DTraceJSDT::pd_dispose() が実際のプローブの追加／削除を行う関数.
/dev/dtrace/helper というデバイスファイル(古い環境では /devices/pseudo/dtrace@0:helper) に対して
ioctl() で DTRACEHIOC_ADDDOF や DTRACEHIOC_REMOVE を指定することで追加/削除を行っている.

Java プログラムから使う場合のエントリポイントは sun.tracing.dtrace.JVM クラス.
このクラスのネイティブメソッドが CVMI 経由で DTraceJSDT を呼び出している.

## 備考(Notes)
CVMI 関数のルックアップは lookupJvmSymbols() 関数で行われる.

ルックアップ結果は jvm_symbols という大域変数に納められており, 
sun.tracing.dtrace.JVM クラスのネイティブメソッドからは, 
この大域変数経由で間接的に CVMI 関数を呼び出している.


```cpp
    ((cite: jdk/src/share/native/sun/tracing/dtrace/JVM.c))
    static JvmSymbols* jvm_symbols = NULL;
```

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
sun.tracing.dtrace.JVM.activate0()
-&gt; JVM_DTraceActivate()
   -&gt; DTraceJSDT::activate()
      -&gt; RegisteredProbes::RegisteredProbes()
      -&gt; ...
      -&gt; AdapterHandlerLibrary::create_dtrace_nmethod()
      -&gt; DTraceJSDT::pd_activate()
      -&gt; RegisteredProbes::toOpaqueProbes()
</pre></div>

<div class="flow-abst"><pre>
sun.tracing.dtrace.JVM.dispose0()
-&gt; JVM_DTraceDispose()
   -&gt; DTraceJSDT::dispose()
      -&gt; RegisteredProbes::toRegisteredProbes()
      -&gt; DTraceJSDT::pd_dispose()
</pre></div>

<div class="flow-abst"><pre>
sun.tracing.dtrace.JVM.isEnabled0()
-&gt; JVM_DTraceIsProbeEnabled()
   -&gt; DTraceJSDT::is_probe_enabled()
      -&gt; NativeInstruction::is_dtrace_trap()
</pre></div>

<div class="flow-abst"><pre>
sun.tracing.dtrace.JVM.isSupported0()
-&gt; JVM_DTraceIsSupported()
   -&gt; DTraceJSDT::is_supported()
      -&gt; DTraceJSDT::pd_is_supported()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






