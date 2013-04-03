---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： GetCurrentContendedMonitor() の処理  
---
[Up](no_DXQUxpU.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： GetCurrentContendedMonitor() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
JvmtiEnv::GetCurrentContendedMonitor()
-> * 対象のスレッドがサスペンドしている場合:
     -> JvmtiEnvBase::get_current_contended_monitor()
   * 〃 がサスペンドしていない場合:
     -> VMThread::execute()
        -> (略) (See: [here](no2935qaz.html) for details)
           -> VM_GetCurrentContendedMonitor::doit()
              -> JvmtiEnvBase::get_current_contended_monitor()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::GetCurrentContendedMonitor()
See: [here](no2935deV.html) for details
### JvmtiEnvBase::get_current_contended_monitor()
(#Under Construction)

### VM_GetCurrentContendedMonitor::doit()
See: [here](no2935qob.html) for details






