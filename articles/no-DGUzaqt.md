---
layout: default
title: CppInterpreterGenerator クラス 
---
[Top](../index.html)

#### CppInterpreterGenerator クラス 



---
## <a name="noONAMzrW8" id="noONAMzrW8">CppInterpreterGenerator</a>

### 概要(Summary)
C++ Interpreter 用のクラス (#ifdef CC_INTERP 時にしか定義されない).

CppInterpreter を生成するための InterpreterGenerator クラス 
(AbstractInterpreterGenerator クラスのサブクラスの1つ).
このクラスによって CppInterpreter クラス用のマシン語コード片が生成される (See: [here](no7882AgC.html) for details).

(なお, コメントでは "template interpreter" と書かれているが typo のような...)


```
    ((cite: hotspot/src/share/vm/interpreter/cppInterpreterGenerator.hpp))
    // This file contains the platform-independent parts
    // of the template interpreter generator.
    
    #ifdef CC_INTERP
    
    class CppInterpreterGenerator: public AbstractInterpreterGenerator {
```

### 備考(Notes)
実際に使用されるのはサブクラスの InterpreterGenerator クラス.




### 詳細(Details)
See: [here](../doxygen/classCppInterpreterGenerator.html) for details

---
