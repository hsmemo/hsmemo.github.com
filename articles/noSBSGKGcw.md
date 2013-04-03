---
layout: default
title: TemplateInterpreterGenerator クラス 
---
[Top](../index.html)

#### TemplateInterpreterGenerator クラス 



---
## <a name="nohkqt_6aC" id="nohkqt_6aC">TemplateInterpreterGenerator</a>

### 概要(Summary)
Template Interpreter 用のクラス (#ifndef CC_INTERP 時にしか定義されない).

TemplateInterpreter を生成するための InterpreterGenerator クラス
(AbstractInterpreterGenerator クラスのサブクラスの1つ).
このクラスによって TemplateInterpreter クラス用のマシン語コード片が生成される (See: [here](no7882AgC.html) for details).


```
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreterGenerator.hpp))
    // This file contains the platform-independent parts
    // of the template interpreter generator.
    
    #ifndef CC_INTERP
    
    class TemplateInterpreterGenerator: public AbstractInterpreterGenerator {
```

### 内部構造(Internal structure)
実際のメソッド定義は hotspot/src/share/vm/interpreter/templateInterpreter.cpp と cpu/ 下のアーキテクチャ依存部にある.

### 備考(Notes)
実際に使用されるのはサブクラスの InterpreterGenerator クラス.




### 詳細(Details)
See: [here](../doxygen/classTemplateInterpreterGenerator.html) for details

---
