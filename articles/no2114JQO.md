---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： hprof 形式のヒープダンプ機能 (com.sun.management.HotSpotDiagnosticMXBean.dumpHeap() の処理) 
---
[Up](noKcyTi5Ec.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： hprof 形式のヒープダンプ機能 (com.sun.management.HotSpotDiagnosticMXBean.dumpHeap() の処理) 

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
sun.management.HotSpotDiagnostic.dumpHeap()
-&gt; Java_sun_management_HotSpotDiagnostic_dumpHeap()
   -&gt; jmm_DumpHeap0()
      -&gt; HeapDumper::dump()
         -&gt; VM_HeapDumper::doit()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### Java_sun_management_HotSpotDiagnostic_dumpHeap()
See: [here](no2114jdm.html) for details
### jmm_DumpHeap0()
See: [here](no2114wns.html) for details
### HeapDumper::dump()
See: [here](no21149xy.html) for details
### VM_HeapDumper::doit()
See: [here](no2114v7B.html) for details






