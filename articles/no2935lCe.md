---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： GenerateEvents() の処理  
---
[Up](nouBV61_3r.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： GenerateEvents() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
JvmtiEnv::GenerateEvents()
-> * event_type 引数が JVMTI_EVENT_COMPILED_METHOD_LOAD の場合:
     -> JvmtiCodeBlobEvents::generate_compiled_method_load_events()
        -> JvmtiExport::post_compiled_method_load()
           -> JvmtiCompiledMethodLoadEventMark::JvmtiCompiledMethodLoadEventMark()
              -> JvmtiCodeBlobEvents::build_jvmti_addr_location_map()
   * event_type 引数が JVMTI_EVENT_DYNAMIC_CODE_GENERATED の場合:
     -> JvmtiCodeBlobEvents::generate_dynamic_code_events()
        -> CodeBlobCollector::collect()
           -> CodeCache::blobs_do(void f(CodeBlob* cb))() (<= 引数は CodeBlobCollector::do_blob())
              -> CodeBlobCollector::do_blob()
        -> JvmtiExport::post_dynamic_code_generated(JvmtiEnv* env, const char *name, const void *code_begin, const void *code_end)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::GenerateEvents()
See: [here](no52482NY.html) for details
### JvmtiCodeBlobEvents::generate_compiled_method_load_events()
See: [here](no2935yMk.html) for details
### nmethod::get_and_cache_jmethod_id()
See: [here](no2935_Wq.html) for details
### methodOopDesc::jmethod_id()
See: [here](no2935Mhw.html) for details
### instanceKlass::get_jmethod_id()
(#Under Construction)
See: [here](no2935Zr2.html) for details
### JvmtiExport::post_compiled_method_load()
See: [here](no17119cAX.html) for details
### JvmtiCompiledMethodLoadEventMark::JvmtiCompiledMethodLoadEventMark()
See: [here](no17119pKd.html) for details
### JvmtiCodeBlobEvents::build_jvmti_addr_location_map()
(#Under Construction)
See: [here](no2935L1F.html) for details
### JvmtiCompiledMethodLoadEventMark::~JvmtiCompiledMethodLoadEventMark()
See: [here](no5248DYe.html) for details

### JvmtiCodeBlobEvents::generate_dynamic_code_events()
See: [here](no17119CsK.html) for details
### CodeBlobCollector::collect()
See: [here](no17119ClW.html) for details
### CodeBlobCollector::do_blob()
See: [here](no171191hE.html) for details
### JvmtiExport::post_dynamic_code_generated(JvmtiEnv* env, const char *name, const void *code_begin, const void *code_end)
See: [here](no2935Y_L.html) for details






