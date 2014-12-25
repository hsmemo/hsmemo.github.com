---
layout: default
title: GCNotifier クラス及びその補助クラス (GCNotificationRequest, GCNotifier)
---
[Top](../index.html)

#### GCNotifier クラス及びその補助クラス (GCNotificationRequest, GCNotifier)

これらは, Platform MXBean 機能のためのクラス.
より具体的に言うと, sun.management.GarbageCollectorImpl.addNotificationListener() メソッドの実装を担当するクラス.
(See: [here](no2114twV.html) and [here](no2114KPr.html) for details)


### クラス一覧(class list)

  * [GCNotifier](#no3rxoDJm7)
  * [GCNotificationRequest](#no3RtZUd7m)


---
## <a name="no3rxoDJm7" id="no3rxoDJm7">GCNotifier</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: sun.management.GarbageCollectorImpl).
(See: [here](no2114KPr.html) for details)

sun.management.GarbageCollectorImpl クラスの通知機能に関する関数を納めた名前空間(AllStatic クラス)


```cpp
    ((cite: hotspot/src/share/vm/services/gcNotifier.hpp))
    class GCNotifier : public AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classGCNotifier.html) for details

---
## <a name="no3RtZUd7m" id="no3RtZUd7m">GCNotificationRequest</a>

### 概要(Summary)
GCNotifier クラス内で使用される補助クラス
(See: [here](no2114KPr.html) for details).

sun.management.GarbageCollectorImpl クラスの通知処理は, 
VMThread が通知の必要性を検出し, ServiceThread によって実際の通知処理が行われる.
この際, VMThread から ServiceThread へは GCNotificationRequest オブジェクトという形で通知内容が伝達される.
1つの GCNotificationRequest オブジェクトが 1つの通知に対応する.

(なお, 最終的には GCNotificationRequest オブジェクト内の情報を基にして ServiceThread が
com.sun.management.GcInfo オブジェクトが生成し, これが登録しているリスナーに通知される)


```cpp
    ((cite: hotspot/src/share/vm/services/gcNotifier.hpp))
    class GCNotificationRequest : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classGCNotificationRequest.html) for details

---
