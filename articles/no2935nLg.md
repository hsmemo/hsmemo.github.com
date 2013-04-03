---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 拡張機能機構 (Extension Mechanism) ： GetExtensionFunctions(), GetExtensionEvents(), SetExtensionEventCallback() の処理  
---
[Up](nohaLVGLDe.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 拡張機能機構 (Extension Mechanism) ： GetExtensionFunctions(), GetExtensionEvents(), SetExtensionEventCallback() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
### 初期化処理
```
(See: [here](no30592Ee.html) for details)
-> JvmtiEnvBase::globally_initialize()
   -> JvmtiExtensions::register_extensions()
```

### GetExtensionFunctions() の処理
```
JvmtiEnv::GetExtensionFunctions()
-> JvmtiExtensions::get_functions()
```

### GetExtensionEvents() の処理
```
JvmtiEnv::GetExtensionEvents()
-> JvmtiExtensions::get_events()
```

### SetExtensionEventCallback() の処理
```
JvmtiEnv::SetExtensionEventCallback()
-> JvmtiExtensions::set_event_callback()
   -> JvmtiEventController::set_extension_event_callback()
      -> JvmtiEventControllerPrivate::set_extension_event_callback()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiExtensions::register_extensions()
See: [here](no7992DOG.html) for details
### JvmtiEnv::GetExtensionFunctions()
See: [here](no2935Bgs.html) for details
### JvmtiExtensions::get_functions()
(#Under Construction)
See: [here](no2935N-H.html) for details
### JvmtiEnv::GetExtensionEvents()
See: [here](no2935Oqy.html) for details
### JvmtiExtensions::get_events()
(#Under Construction)
See: [here](no2935aIO.html) for details
### JvmtiEnv::SetExtensionEventCallback()
See: [here](no2935A0B.html) for details
### JvmtiExtensions::set_event_callback()
See: [here](no2935nSU.html) for details
### JvmtiEventController::set_extension_event_callback()
See: [here](no2935aPC.html) for details
### JvmtiEventControllerPrivate::set_extension_event_callback()
(#Under Construction)
See: [here](no2935nZI.html) for details






