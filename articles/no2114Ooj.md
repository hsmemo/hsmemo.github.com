---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.ClassLoadingMXBean 
---
[Up](nouYTgvZOF.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： java.lang.management.ClassLoadingMXBean 

--- 
## 概要(Summary)
クラスのロード／アンロード処理が行われるたびに ClassLoadingService オブジェクト内の PerfData に統計情報が蓄えられていく.

sun.management.ClassLoadingImpl クラスのメソッドは, 単にその統計情報を取得するだけ.

## 処理の流れ (概要)(Execution Flows : Summary)
### 統計情報の更新処理
```
ClassFileParser::parseClassFile()
-> ClassLoadingService::notify_class_loaded()
```

```
ClassFileParser::parse_method()
-> ClassLoadingService::add_class_method_size()
```

```
SystemDictionary::load_shared_class()
-> ClassLoadingService::notify_class_loaded()
```

```
Dictionary::do_unloading()
-> ClassLoadingService::notify_class_unloaded()
```

### sun.management.ClassLoadingImpl クラスのメソッドの処理 (統計情報の取得処理)
```
sun.management.ClassLoadingImpl.isVerbose()
-> sun.management.VMManagementImpl.getVerboseClass()
   -> Java_sun_management_VMManagementImpl_getVerboseClass()
      -> jmm_GetBoolAttribute()  (JMM_VERBOSE_CLASS を引数として呼び出される)
         -> ClassLoadingService::get_verbose()
```

```
sun.management.ClassLoadingImpl.setVerboseClass()
-> Java_sun_management_ClassLoadingImpl_setVerboseClass()
   -> jmm_SetBoolAttribute()  (JMM_VERBOSE_CLASS を引数として呼び出される)
      -> ClassLoadingService::set_verbose()
```

```
sun.management.ClassLoadingImpl.getTotalLoadedClassCount()
-> sun.management.VMManagementImpl.getTotalClassCount()
   -> Java_sun_management_VMManagementImpl_getTotalClassCount()
      -> jmm_GetLongAttribute()  (JMM_CLASS_LOADED_COUNT を引数として呼び出される)
         -> get_long_attribute()
            -> ClassLoadingService::loaded_class_count()
```

```
sun.management.ClassLoadingImpl.getUnloadedClassCount()
-> sun.management.VMManagementImpl.getUnloadedClassCount()
   -> Java_sun_management_VMManagementImpl_getUnloadedClassCount()
      -> jmm_GetLongAttribute()  (JMM_CLASS_UNLOADED_COUNT を引数として呼び出される)
         -> get_long_attribute()
            -> ClassLoadingService::unloaded_class_count()
```

```
sun.management.ClassLoadingImpl.getLoadedClassCount()
-> sun.management.VMManagementImpl.getLoadedClassCount()
   -> sun.management.VMManagementImpl.getTotalClassCount()
      -> (同上)
   -> sun.management.VMManagementImpl.getUnloadedClassCount()
      -> (同上)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### ClassLoadingService::notify_class_loaded()
See: [here](no21140aL.html) for details
### ClassLoadingService::add_class_method_size()
See: [here](no2114OvX.html) for details
### ClassLoadingService::notify_class_unloaded()
See: [here](no2114BlR.html) for details
### sun.management.ClassLoadingImpl.isVerbose()
See: [here](no2114b5d.html) for details
### Java_sun_management_VMManagementImpl_getVerboseClass()
See: [here](no2114BsF.html) for details
### jmm_GetBoolAttribute()
See: [here](no2114bHG.html) for details
### ClassLoadingService::get_verbose()
See: [here](no2114bAS.html) for details
### Java_sun_management_ClassLoadingImpl_setVerboseClass()
See: [here](no2114oKY.html) for details
### ClassLoadingService::set_verbose()
See: [here](no21141Ue.html) for details
### ClassLoadingService::reset_trace_class_unloading()
See: [here](no2114Cfk.html) for details
### sun.management.ClassLoadingImpl.getTotalLoadedClassCount()
See: [here](no2114oDk.html) for details
### Java_sun_management_VMManagementImpl_getTotalClassCount()
See: [here](no2114O2L.html) for details
### jmm_GetLongAttribute()
See: [here](no2114oRM.html) for details
### get_long_attribute()
See: [here](no21141bS.html) for details
### ClassLoadingService::loaded_class_count()
See: [here](no2114Ppq.html) for details
### sun.management.ClassLoadingImpl.getUnloadedClassCount()
See: [here](no2114CYw.html) for details
### Java_sun_management_VMManagementImpl_getUnloadedClassCount()
See: [here](no2114Pi2.html) for details
### ClassLoadingService::unloaded_class_count()
See: [here](no2114CmY.html) for details
### sun.management.ClassLoadingImpl.getLoadedClassCount()
See: [here](no21141Nq.html) for details
### sun.management.VMManagementImpl.getLoadedClassCount()
See: [here](no2114czw.html) for details






