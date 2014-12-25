---
layout: default
title: Compile::Output() 用の補助クラス (Scheduling, NonSafepointEmitter)
---
[Top](../index.html)

#### Compile::Output() 用の補助クラス (Scheduling, NonSafepointEmitter)

これらは, C2 JIT Compiler 用のクラス.
より具体的に言うと, マシン語を出力する処理(Compile::Output()の処理)で使用される補助クラス.


### クラス一覧(class list)

  * [Scheduling](#nohnVSPGQI)
  * [NonSafepointEmitter](#noL_d7oFZw)


---
## <a name="nohnVSPGQI" id="nohnVSPGQI">Scheduling</a>

### 概要(Summary)
Compile クラス内で使用される補助クラス.

低レベル中間語に対して命令レベルのスケジューリングを行う.

(なお, このクラスは StackObjクラスではないが現状では局所変数としてのみ生成されている)


```cpp
    ((cite: hotspot/src/share/vm/opto/output.hpp))
    //------------------------------Scheduling----------------------------------
    // This class contains all the information necessary to implement instruction
    // scheduling and bundling.
    class Scheduling {
```

### 使われ方(Usage)
Compile::ScheduleAndBundle() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Compile::Output()
-> Compile::ScheduleAndBundle()
```




### 詳細(Details)
See: [here](../doxygen/classScheduling.html) for details

---
## <a name="noL_d7oFZw" id="noL_d7oFZw">NonSafepointEmitter</a>

### 概要(Summary)
Compile クラス内で使用される補助クラス.

safepoint ではない箇所のメタ情報を DebugInformationRecorder に記録するためのクラス.

(なお, このクラスは StackObjクラスではないが現状では局所変数としてのみ生成されている)


```cpp
    ((cite: hotspot/src/share/vm/opto/output.cpp))
    // A simplified version of Process_OopMap_Node, to handle non-safepoints.
    class NonSafepointEmitter {
```

### 使われ方(Usage)
Compile::Fill_buffer() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
Compile::Output()
-> Compile::Fill_buffer()
```





### 詳細(Details)
See: [here](../doxygen/classNonSafepointEmitter.html) for details

---
