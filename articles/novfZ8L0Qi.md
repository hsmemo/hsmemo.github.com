---
layout: default
title: ClassLoadingService クラス関連のクラス (ClassLoadingService, LoadedClassesEnumerator)
---
[Top](../index.html)

#### ClassLoadingService クラス関連のクラス (ClassLoadingService, LoadedClassesEnumerator)

これらは, Platform MXBean 機能のためのクラス.
より具体的に言うと, sun.management.ClassLoadingImpl クラス及び sun.management.HotspotClassLoading クラスの実装を担当するクラス.
(See: [here](no2114twV.html) for details)
(See: [here](no2114Ooj.html) and [here](no2114pLf.html) for details)

(ついでに, (DTrace のフック点) を提供する役割もある)


### クラス一覧(class list)

  * [ClassLoadingService](#noNcRAjMxV)
  * [LoadedClassesEnumerator](#no7DpSrjvB)


---
## <a name="noNcRAjMxV" id="noNcRAjMxV">ClassLoadingService</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: sun.management.ClassLoadingImpl, sun.management.HotspotClassLoadingMBean).
(See: [here](no2114Ooj.html) and [here](no2114pLf.html) for details)

クラスファイルのロード処理に関する PerfData を納めた名前空間(AllStatic クラス).

(PerfData に記録している内容は, クラスファイルがロード／アンロードされた回数, その際のロード／アンロードされたバイト数, 等)


```cpp
    ((cite: hotspot/src/share/vm/services/classLoadingService.hpp))
    // VM monitoring and management support for the Class Loading subsystem
    class ClassLoadingService : public AllStatic {
```

### 使われ方(Usage)
sun.management.ClassLoadingImpl クラス及び sun.management.HotspotClassLoading クラスから(のみ)使用されている.

(<= ただし内部に持っているのは PerfData なので jcmd 等から見ようと思えば見ることはできるが...)

### 内部構造(Internal structure)
内部には以下のような Perf データを格納している.

```cpp
    ((cite: hotspot/src/share/vm/services/classLoadingService.hpp))
      // Counters for classes loaded from class files
      static PerfCounter*  _classes_loaded_count;
      static PerfCounter*  _classes_unloaded_count;
      static PerfCounter*  _classbytes_loaded;
      static PerfCounter*  _classbytes_unloaded;
    
      // Counters for classes loaded from shared archive
      static PerfCounter*  _shared_classes_loaded_count;
      static PerfCounter*  _shared_classes_unloaded_count;
      static PerfCounter*  _shared_classbytes_loaded;
      static PerfCounter*  _shared_classbytes_unloaded;
    
      static PerfVariable* _class_methods_size;
```

これらの Perf データには, それぞれ以下の名前でアクセス可能.

  * java.cls.loadedClasses
  * java.cls.unloadedClasses
  * java.cls.sharedLoadedClasses
  * java.cls.sharedUnloadedClasses
  * sun.cls.loadedBytes
  * sun.cls.unloadedBytes
  * sun.cls.sharedLoadedBytes
  * sun.cls.sharedUnloadedBytes
  * sun.cls.methodBytes

```cpp
    ((cite: hotspot/src/share/vm/services/classLoadingService.cpp))
      // These counters are for java.lang.management API support.
      // They are created even if -XX:-UsePerfData is set and in
      // that case, they will be allocated on C heap.
      _classes_loaded_count =
                     PerfDataManager::create_counter(JAVA_CLS, "loadedClasses",
                                                     PerfData::U_Events, CHECK);
    
      _classes_unloaded_count =
                     PerfDataManager::create_counter(JAVA_CLS, "unloadedClasses",
                                                     PerfData::U_Events, CHECK);
    
      _shared_classes_loaded_count =
                     PerfDataManager::create_counter(JAVA_CLS, "sharedLoadedClasses",
                                                     PerfData::U_Events, CHECK);
    
      _shared_classes_unloaded_count =
                     PerfDataManager::create_counter(JAVA_CLS, "sharedUnloadedClasses",
                                                     PerfData::U_Events, CHECK);
    
      if (UsePerfData) {
        _classbytes_loaded =
                     PerfDataManager::create_counter(SUN_CLS, "loadedBytes",
                                                     PerfData::U_Bytes, CHECK);
    
        _classbytes_unloaded =
                     PerfDataManager::create_counter(SUN_CLS, "unloadedBytes",
                                                     PerfData::U_Bytes, CHECK);
        _shared_classbytes_loaded =
                     PerfDataManager::create_counter(SUN_CLS, "sharedLoadedBytes",
                                                     PerfData::U_Bytes, CHECK);
    
        _shared_classbytes_unloaded =
                     PerfDataManager::create_counter(SUN_CLS, "sharedUnloadedBytes",
                                                     PerfData::U_Bytes, CHECK);
        _class_methods_size =
                     PerfDataManager::create_variable(SUN_CLS, "methodBytes",
                                                      PerfData::U_Bytes, CHECK);
      }
```

### 備考(Notes)
なお, 以下の PerfData は, Java のクラスからは使われていない.

  * _shared_classes_loaded_count
  * _shared_classes_unloaded_count
  * _shared_classbytes_loaded
  * _shared_classbytes_unloaded

一応アクセス用の jmm の関数までは用意されている
(しかしこれらは Java のクラスからは使われていない...).

<div class="flow-abst"><pre>
  -&gt; jmm_GetLongAttribute()
     -&gt; get_long_attribute()  (引数が JMM_SHARED_CLASS_LOADED_COUNT の場合)
        -&gt; ClassLoadingService::loaded_shared_class_count()

  -&gt; jmm_GetLongAttribute()
     -&gt; get_long_attribute()  (引数が JMM_SHARED_CLASS_UNLOADED_COUNT の場合)
        -&gt; ClassLoadingService::unloaded_shared_class_count()

  -&gt; jmm_GetLongAttribute()
     -&gt; get_long_attribute()  (引数が JMM_SHARED_CLASS_LOADED_BYTES の場合)
        -&gt; ClassLoadingService::loaded_shared_class_bytes()

  -&gt; jmm_GetLongAttribute()
     -&gt; get_long_attribute()  (引数が JMM_SHARED_CLASS_UNLOADED_BYTES の場合)
        -&gt; ClassLoadingService::unloaded_shared_class_bytes()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classClassLoadingService.html) for details

---
## <a name="no7DpSrjvB" id="no7DpSrjvB">LoadedClassesEnumerator</a>

### 概要(Summary)
jmm_GetLoadedClasses() 内で使用される補助クラス.
ロード済みのクラス全てをたどるためのイテレータクラス(StackObjクラス).

(なおコメントによると, このコードは JMM と JVMTI で共通化すべき, とのこと)

```cpp
    ((cite: hotspot/src/share/vm/services/classLoadingService.hpp))
    // FIXME: make this piece of code to be shared by M&M and JVMTI
    class LoadedClassesEnumerator : public StackObj {
```

### 使われ方(Usage)
#### 使用例(usage examples)
使用方法は次のような感じ.


```cpp
    ((cite: hotspot/src/share/vm/services/management.cpp))
      LoadedClassesEnumerator lce(THREAD);  // Pass current Thread as parameter
    
      int num_classes = lce.num_loaded_classes();
    ...
      for (int i = 0; i < num_classes; i++) {
        KlassHandle kh = lce.get_klass(i);
    ...
      }
```

#### 使用箇所(where its instances are used)
jmm_GetLoadedClasses() 内で(のみ)使用されている.

(ところでこのクラスは使われているか??
 jmm_GetLoadedClasses() でしか使われてないが, jmm_GetLoadedClasses() 自体がどこからも呼ばれていないような... #TODO)
(なお, 似た名前の関数として JvmtiEnv::GetLoadedClasses() というものがあるが, これは関係ない模様)




### 詳細(Details)
See: [here](../doxygen/classLoadedClassesEnumerator.html) for details

---
