---
layout: default
title: JvmtiManageCapabilities クラス 
---
[Top](../index.html)

#### JvmtiManageCapabilities クラス 



---
## <a name="no86ROvftI" id="no86ROvftI">JvmtiManageCapabilities</a>

### 概要(Summary)
JVMTI の機能を実現するためのクラス. 
より具体的に言うと, JVMTI の capability(権限)管理用の関数を納めた名前空間(AllStatic クラス) (See: [here](no2935trw.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiManageCapabilities.hpp))
    class JvmtiManageCapabilities : public AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JvmtiEnv の capability 処理用のメソッド内
  (JvmtiEnv::GetPotentialCapabilities(), JvmtiEnv::AddCapabilities(),
  JvmtiEnv::RelinquishCapabilities(), JvmtiEnv::GetCapabilities())

  これらは同名の JVMTI 関数を実現するためのメソッド (See: [here](no2935trw.html) for details).

* JvmtiEnvBase::globally_initialize()

  これは JVMTI の GetEnv() 関数を実現するためのメソッド (See: [here](no30592Ee.html) for details).

* JvmtiEnvBase::env_dispose()

  これは JVMTI の DisposeEnvironment() 関数を実現するためのメソッド (See: [here](no2935HVZ.html) for details).

* JvmtiExport::enter_start_phase()

  これは HotSpot の起動中に Threads::create_vm() 内で呼び出されるメソッド.




### 詳細(Details)
See: [here](../doxygen/classJvmtiManageCapabilities.html) for details

---
