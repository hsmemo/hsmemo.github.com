---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： メソッド (Method) ： GetBytecodes() の処理  
---
[Up](nofo_1oJGp.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： メソッド (Method) ： GetBytecodes() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
JvmtiEnv::GetBytecodes()
-> JvmtiEnvBase::allocate()
-> JvmtiClassFileReconstituter::copy_bytecodes()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::GetBytecodes()
See: [here](no17119Qwj.html) for details
### JvmtiClassFileReconstituter::copy_bytecodes()
See: [here](no17119qEw.html) for details






