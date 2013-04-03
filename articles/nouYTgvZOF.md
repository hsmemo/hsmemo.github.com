---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理
---
[Up](no2114S_x.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理

--- 
(#Under Construction)

## 備考(Notes)
sun.management パッケージ下にある Platform MXBeal の実装クラス群からは,
sun.management.VMManagement インターフェースを介して HotSpot 内部にアクセスが行われている
(sun.management.VMManagement はインターフェースであり, 実装している実体は sun.management.VMManagementImpl).

ただし, 中には直接 HotSpot 内部にアクセスするものもある.
VMManagement では is*() とか get*() みたいな値を参照するだけのものを扱い,
set*() みたいな状態を変更する機能については直接アクセスしている模様.
jmm_interface 変数を直接使っているクラスは以下の通り.

* sun.management.ClassLoadingImpl
* sun.management.Flag
* sun.management.GarbageCollectorImpl
* sun.management.GcInfoBuilder
* sun.management.HotSpotDiagnostic
* sun.management.HotspotThread
* sun.management.MemoryImpl
* sun.management.MemoryManagerImpl
* sun.management.MemoryPoolImpl
* sun.management.ThreadImpl
* sun.management.VMManagementImpl




## Subcategories
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.ClassLoadingMXBean ](no2114Ooj.html)
* [(#TBD) Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.CompilationMXBean](no9l1uVGx5.html)
* [(#TBD) Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.MemoryMXBean](noaM_oMJxs.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.MemoryPoolMXBean](nozwu62341.html)
* [(#TBD) Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.MemoryManagerMXBean](noyB9ISFtv.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.GarbageCollectorMXBean](noCxwFq9be.html)
* [(#TBD) Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.OperatingSystemMXBean](nogOkxJWY0.html)
* [(#TBD) Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.RuntimeMXBean](nobu03DWt7.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.ThreadMXBean](noMz-1isvk.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： com.sun.management.HotSpotDiagnosticMXBean](noKcyTi5Ec.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotClassLoadingMBean ](no2114pLf.html)
* [(#TBD) Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotCompilationMBean](noUwYt01ta.html)
* [(#TBD) Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotInternalMBean](noXM136k-I.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotMemoryMBean](norvN2FPOq.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotRuntimeMBean ](no2114byp.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotThreadMBean  ](no2114vml.html)



