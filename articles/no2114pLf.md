---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotClassLoadingMBean 
---
[Up](nouYTgvZOF.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotClassLoadingMBean 

--- 
## 概要(Summary)
sun.management.HotspotClassLoading クラスの各メソッドは,
ClassLoadingService クラスまたは ClassLoader クラス内の値を取得するか,
あるいはクラスローディングに関連した PerfData の値を取得するだけ.

## 備考(Notes)
(なお, このクラスは JSR-174 には存在しない Sun Microsystems の独自拡張機能)

## 処理の流れ (概要)(Execution Flows : Summary)
### sun.management.HotspotClassLoadingMBean.getLoadedClassSize() の処理
```
sun.management.HotspotClassLoading.getLoadedClassSize()
-> sun.management.VMManagementImpl.getLoadedClassSize()
   -> Java_sun_management_VMManagementImpl_getLoadedClassSize()
      -> jmm_GetLongAttribute()  (JMM_CLASS_LOADED_BYTES を引数として呼び出される)
         -> get_long_attribute()
            -> ClassLoadingService::loaded_class_bytes()
```

### sun.management.HotspotClassLoadingMBean.getUnloadedClassSize() の処理
```
sun.management.HotspotClassLoading.getUnloadedClassSize()
-> sun.management.VMManagementImpl.getUnloadedClassSize()
   -> Java_sun_management_VMManagementImpl_getUnloadedClassSize()
      -> jmm_GetLongAttribute()  (JMM_CLASS_UNLOADED_BYTES を引数として呼び出される)
         -> get_long_attribute()
            -> ClassLoadingService::unloaded_class_bytes()
```

### sun.management.HotspotClassLoadingMBean.getMethodDataSize() の処理
```
sun.management.HotspotClassLoading.getMethodDataSize()
-> sun.management.VMManagementImpl.getMethodDataSize()
   -> Java_sun_management_VMManagementImpl_getMethodDataSize()
      -> jmm_GetLongAttribute()  (JMM_METHOD_DATA_SIZE_BYTES を引数として呼び出される)
         -> get_long_attribute()
            -> ClassLoadingService::class_method_data_size()
```

### ... (#TODO)


## 処理の流れ (詳細)(Execution Flows : Details)
### sun.management.HotspotClassLoading.getLoadedClassSize()
See: [here](no2114CtM.html) for details
### Java_sun_management_VMManagementImpl_getLoadedClassSize()
See: [here](no21141iG.html) for details
### ClassLoadingService::loaded_class_bytes()
See: [here](no2114Pwe.html) for details

### sun.management.HotspotClassLoading.getUnloadedClassSize()
See: [here](no2114P3S.html) for details
### Java_sun_management_VMManagementImpl_getUnloadedClassSize()
See: [here](no2114oYA.html) for details
### ClassLoadingService::unloaded_class_bytes()
See: [here](no2114c6k.html) for details

### sun.management.HotspotClassLoading.getMethodDataSize()
See: [here](no2114cBZ.html) for details
### Java_sun_management_VMManagementImpl_getMethodDataSize()
See: [here](no21142Ox.html) for details
### ClassLoadingService::class_method_data_size()
See: [here](no2114pEr.html) for details





