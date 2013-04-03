---
layout: default
title: HeapInspection クラス及びその補助クラス (KlassInfoEntry, KlassInfoClosure, KlassInfoBucket, KlassInfoTable, KlassInfoHisto, HeapInspection, 及びそれらの補助クラス(HistoClosure, RecordInstanceClosure, FindInstanceClosure))
---
[Top](../index.html)

#### HeapInspection クラス及びその補助クラス (KlassInfoEntry, KlassInfoClosure, KlassInfoBucket, KlassInfoTable, KlassInfoHisto, HeapInspection, 及びそれらの補助クラス(HistoClosure, RecordInstanceClosure, FindInstanceClosure))

これらは, 保守運用機能のためのクラス (関連する serviceability 機能からのみ使用される).


### クラス一覧(class list)

  * [HeapInspection](#nonmuQ_yXf)
  * [KlassInfoEntry](#nolkEl9Z5y)
  * [KlassInfoClosure](#no_Mph77wz)
  * [KlassInfoBucket](#noJ26C4LmG)
  * [KlassInfoTable](#noMRuUePQI)
  * [KlassInfoHisto](#nojwxivRiu)
  * [HistoClosure](#no3fqo5DZ1)
  * [RecordInstanceClosure](#noDer8dmbi)
  * [FindInstanceClosure](#noBiMH7yXy)


---
## <a name="nonmuQ_yXf" id="nonmuQ_yXf">HeapInspection</a>

### 概要(Summary)
Java ヒープ内に存在する Java オブジェクト(インスタンス)の情報を調べるクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
    class HeapInspection : public AllStatic {
```

以下の2つの関数(のみ)が定義されている.

* HeapInspection::heap_inspection() : 
  「ヒープ内にどのクラスのインスタンスがどれだけ(何個および何バイト)存在しているか」という情報を出力する

* HeapInspection::find_instances_at_safepoint() : 
  ヒープ中に存在する指定されたクラスのインスタンス(やそのサブクラスのインスタンス)全てを取得する
  (結果は GrowableArray に格納されて返される)


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
      static void heap_inspection(outputStream* st, bool need_prologue) KERNEL_RETURN;
      static void find_instances_at_safepoint(klassOop k, GrowableArray<oop>* result) KERNEL_RETURN;
```

### 使われ方(Usage)
それぞれ, 以下の関数内で(のみ)呼び出されている.

* HeapInspection::heap_inspection() :
  VM_GC_HeapInspection::doit()

  (なお, VM_GC_HeapInspection は「どのインスタンスがどれだけ存在しているか」という情報を出力する VM Operation.
   (See: VM_GC_HeapInspection))

* HeapInspection::find_instances_at_safepoint() :
  ConcurrentLocksDump::dump_at_safepoint()

  (ConcurrentLocksDump::dump_at_safepoint() は「現在ロックしているシンクロナイザの一覧」を取得する関数.
  VM_PrintThreads::doit() 内 (正確にはその中で呼び出される Threads::print_on() 内) と
  VM_ThreadDump::doit() 内で(のみ)呼び出されている.
  (See: VM_PrintThreads, VM_ThreadDump))

#### 参考(for your information): VM_GC_HeapInspection::doit()
See: [here](no28916Gaj.html) for details
#### 参考(for your information): VM_PrintThreads::doit()
See: [here](no28916t_p.html) for details
#### 参考(for your information): Threads::print_on()
See: [here](no28916vPg.html) for details
#### 参考(for your information): VM_ThreadDump::doit()
See: [here](no2114QGC.html) for details
### 内部構造(Internal structure)
定義されているメソッドはそれぞれ以下の通り.

#### 参考(for your information): HeapInspection::heap_inspection()
See: [here](no7882iKv.html) for details
#### 参考(for your information): HeapInspection::find_instances_at_safepoint()
See: [here](no78827rc.html) for details

### 備考(Notes)
(HeapInspection::heap_inspection() による具体的な表示内容が知りたければ,
-XX:+PrintClassHistogram 付きで java を実行し, SIGQUIT を投げてみればいい)




### 詳細(Details)
See: [here](../doxygen/classHeapInspection.html) for details

---
## <a name="nolkEl9Z5y" id="nolkEl9Z5y">KlassInfoEntry</a>

### 概要(Summary)
HeapInspection::heap_inspection() 内で使用される補助クラス.

「あるクラスのインスタンスがどれだけ(何個および何バイト)存在しているか」という情報を格納する.


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
    class KlassInfoEntry: public CHeapObj {
```

### 内部構造(Internal structure)
内部には以下の4つのフィールド(のみ)を含む.

* _next : (KlassInfoBucket 内でリスト状に管理される際に使用されるフィールド) 次のオブジェクトを指すポインタ
* _klass : この KlassInfoEntry が対応するクラス
* _instance_count : そのクラスのインスタンスが何個存在しているか
* _instance_words : そのクラスのインスタンスが何バイト存在しているか

```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
      KlassInfoEntry* _next;
      klassOop        _klass;
      long            _instance_count;
      size_t          _instance_words;
```




### 詳細(Details)
See: [here](../doxygen/classKlassInfoEntry.html) for details

---
## <a name="no_Mph77wz" id="no_Mph77wz">KlassInfoClosure</a>

### 概要(Summary)
HeapInspection::heap_inspection() の処理用の補助クラス.

KlassInfoEntry へのポインタに対して何らかの処理を行う Closure クラスの基底クラス.

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
    class KlassInfoClosure: public StackObj {
```

### 使われ方(Usage)
KlassInfoEntry* を処理する do_cinfo() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
      // Called for each KlassInfoEntry.
      virtual void do_cinfo(KlassInfoEntry* cie) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classKlassInfoClosure.html) for details

---
## <a name="noJ26C4LmG" id="noJ26C4LmG">KlassInfoBucket</a>

### 概要(Summary)
KlassInfoTable クラス内で使用される補助クラス.

KlassInfoEntry オブジェクトを格納するための線形リスト
(KlassInfoTable はこのクラスを用いてハッシュテーブルを構築する).


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
    class KlassInfoBucket: public CHeapObj {
```


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
    // A KlassInfoBucket is the head of a link list
    // of KlassInfoEntry's
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
KlassInfoTable オブジェクトの _buckets フィールド内に(のみ)格納されている.

```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
      KlassInfoBucket* _buckets;
```

### 内部構造(Internal structure)
以下の _list フィールドに KlassInfoEntry オブジェクトを線形リスト状に格納している.


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
      KlassInfoEntry* _list;
```




### 詳細(Details)
See: [here](../doxygen/classKlassInfoBucket.html) for details

---
## <a name="noMRuUePQI" id="noMRuUePQI">KlassInfoTable</a>

### 概要(Summary)
HeapInspection::heap_inspection() 内で使用される補助クラス.

ヒープ中を辿ってインスタンスの情報を集めていく際に, 集めた情報を格納しておくために使われる.


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
    class KlassInfoTable: public StackObj {
```


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
    // KlassInfoTable is a bucket hash table that
    // maps klassOops to extra information:
    //    instance count and instance word size.
```

### 使われ方(Usage)
HeapInspection::heap_inspection() 内で (のみ) 局所変数として生成されている.

(HeapInspection::heap_inspection() で出力される情報は,
まずこの KlassInfoTable オブジェクト内に収集される)

### 内部構造(Internal structure)
内部には, KlassInfoBucket を用いて構築したハッシュテーブルを持っている
(klassOops をキーとして対応する KlassInfoEntry を引くことが可能).




### 詳細(Details)
See: [here](../doxygen/classKlassInfoTable.html) for details

---
## <a name="nojwxivRiu" id="nojwxivRiu">KlassInfoHisto</a>

### 概要(Summary)
HeapInspection::heap_inspection() 内で使用される補助クラス.

ヒープ中を辿って集めた情報を, ソートして出力する際に使用される.


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
    class KlassInfoHisto : public StackObj {
```


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.hpp))
    // KlassInfoHisto is a growable array of pointers
    // to KlassInfoEntry's and is used to sort
    // the entries.
```

### 使われ方(Usage)
HeapInspection::heap_inspection() 内で (のみ) 局所変数として生成されている.

(KlassInfoTable オブジェクト内に収集された情報は,
KlassInfoHisto に移された後, ソートされて出力される)




### 詳細(Details)
See: [here](../doxygen/classKlassInfoHisto.html) for details

---
## <a name="no3fqo5DZ1" id="no3fqo5DZ1">HistoClosure</a>

### 概要(Summary)
HeapInspection::heap_inspection() 内で使用される補助クラス.

ヒープ中を辿って KlassInfoTable 内に収集された情報を 
KlassInfoHisto に移し替える作業で使われる Closure.


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.cpp))
    class HistoClosure : public KlassInfoClosure {
```




### 詳細(Details)
See: [here](../doxygen/classHistoClosure.html) for details

---
## <a name="noDer8dmbi" id="noDer8dmbi">RecordInstanceClosure</a>

### 概要(Summary)
HeapInspection::heap_inspection() 内で使用される補助クラス.

ヒープ中を辿ってインスタンスの情報を KlassInfoTable 内に収集する Closure.


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.cpp))
    class RecordInstanceClosure : public ObjectClosure {
```




### 詳細(Details)
See: [here](../doxygen/classRecordInstanceClosure.html) for details

---
## <a name="noBiMH7yXy" id="noBiMH7yXy">FindInstanceClosure</a>

### 概要(Summary)
HeapInspection::find_instances_at_safepoint() 内で使用される補助クラス.

ヒープ中を辿って指定されたクラスのインスタンス(やそのサブクラスのインスタンス)を収集する Closure.


```
    ((cite: hotspot/src/share/vm/memory/heapInspection.cpp))
    class FindInstanceClosure : public ObjectClosure {
```




### 詳細(Details)
See: [here](../doxygen/classFindInstanceClosure.html) for details

---
