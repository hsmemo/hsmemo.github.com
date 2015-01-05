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
<div class="flow-abst"><pre>
ClassFileParser::parseClassFile()
-&gt; ClassLoadingService::notify_class_loaded()
</pre></div>

<div class="flow-abst"><pre>
ClassFileParser::parse_method()
-&gt; ClassLoadingService::add_class_method_size()
</pre></div>

<div class="flow-abst"><pre>
SystemDictionary::load_shared_class()
-&gt; ClassLoadingService::notify_class_loaded()
</pre></div>

<div class="flow-abst"><pre>
Dictionary::do_unloading()
-&gt; ClassLoadingService::notify_class_unloaded()
</pre></div>

### sun.management.ClassLoadingImpl クラスのメソッドの処理 (統計情報の取得処理)
<div class="flow-abst"><pre>
sun.management.ClassLoadingImpl.isVerbose()
-&gt; sun.management.VMManagementImpl.getVerboseClass()
   -&gt; Java_sun_management_VMManagementImpl_getVerboseClass()
      -&gt; jmm_GetBoolAttribute()  (JMM_VERBOSE_CLASS を引数として呼び出される)
         -&gt; ClassLoadingService::get_verbose()
</pre></div>

<div class="flow-abst"><pre>
sun.management.ClassLoadingImpl.setVerboseClass()
-&gt; Java_sun_management_ClassLoadingImpl_setVerboseClass()
   -&gt; jmm_SetBoolAttribute()  (JMM_VERBOSE_CLASS を引数として呼び出される)
      -&gt; ClassLoadingService::set_verbose()
</pre></div>

<div class="flow-abst"><pre>
sun.management.ClassLoadingImpl.getTotalLoadedClassCount()
-&gt; sun.management.VMManagementImpl.getTotalClassCount()
   -&gt; Java_sun_management_VMManagementImpl_getTotalClassCount()
      -&gt; jmm_GetLongAttribute()  (JMM_CLASS_LOADED_COUNT を引数として呼び出される)
         -&gt; get_long_attribute()
            -&gt; ClassLoadingService::loaded_class_count()
</pre></div>

<div class="flow-abst"><pre>
sun.management.ClassLoadingImpl.getUnloadedClassCount()
-&gt; sun.management.VMManagementImpl.getUnloadedClassCount()
   -&gt; Java_sun_management_VMManagementImpl_getUnloadedClassCount()
      -&gt; jmm_GetLongAttribute()  (JMM_CLASS_UNLOADED_COUNT を引数として呼び出される)
         -&gt; get_long_attribute()
            -&gt; ClassLoadingService::unloaded_class_count()
</pre></div>

<div class="flow-abst"><pre>
sun.management.ClassLoadingImpl.getLoadedClassCount()
-&gt; sun.management.VMManagementImpl.getLoadedClassCount()
   -&gt; sun.management.VMManagementImpl.getTotalClassCount()
      -&gt; (同上)
   -&gt; sun.management.VMManagementImpl.getUnloadedClassCount()
      -&gt; (同上)
</pre></div>


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






