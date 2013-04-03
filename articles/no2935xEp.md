---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： クラス (Class) ： GetConstantPool() の処理  
---
[Up](noSfr5xs8r.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： クラス (Class) ： GetConstantPool() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
JvmtiEnv::GetConstantPool()
-> JvmtiConstantPoolReconstituter::JvmtiConstantPoolReconstituter()
-> JvmtiEnvBase::allocate()
-> JvmtiConstantPoolReconstituter::copy_cpool_bytes()
-> JvmtiConstantPoolReconstituter::~JvmtiConstantPoolReconstituter()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::GetConstantPool()
See: [here](no17119P9E.html) for details
### JvmtiConstantPoolReconstituter::JvmtiConstantPoolReconstituter()
See: [here](no17119cHL.html) for details
### JvmtiConstantPoolReconstituter::copy_cpool_bytes()
See: [here](no17119pRR.html) for details
### JvmtiConstantPoolReconstituter::~JvmtiConstantPoolReconstituter()
See: [here](no171192bX.html) for details






