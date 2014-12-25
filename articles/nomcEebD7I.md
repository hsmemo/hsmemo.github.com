---
layout: default
title: CppInterpreter クラス 
---
[Top](../index.html)

#### CppInterpreter クラス 



---
## <a name="no-7nwctlD" id="no-7nwctlD">CppInterpreter</a>

### 概要(Summary)
C++ Interpreter 用のクラス (#ifdef CC_INTERP 時にしか定義されない).

C++ Interpreter において Interpreter の初期化や生成されたコードの管理を行うためのクラス
(= C++ Interpreter 用の Interpreter クラス). (See: [here](no7882AgC.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/cppInterpreter.hpp))
    #ifdef CC_INTERP
    
    // This file contains the platform-independent parts
    // of the c++ interpreter
    
    class CppInterpreter: public AbstractInterpreter {
```




### 詳細(Details)
See: [here](../doxygen/classCppInterpreter.html) for details

---
