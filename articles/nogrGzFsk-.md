---
layout: default
title: AbstractInterpreter クラスおよび AbstractInterpreterGenerator クラス (AbstractInterpreter, AbstractInterpreterGenerator)
---
[Top](../index.html)

#### AbstractInterpreter クラスおよび AbstractInterpreterGenerator クラス (AbstractInterpreter, AbstractInterpreterGenerator)



### クラス一覧(class list)

  * [AbstractInterpreter](#noPEZIKMb9)
  * [AbstractInterpreterGenerator](#noaChWlR_l)


---
## <a name="noPEZIKMb9" id="noPEZIKMb9">AbstractInterpreter</a>

### 概要(Summary)
インタープリタ(Interpreter)関連の機能を納めた名前空間(AllStatic クラス).

このクラスが全ての Interpreter クラスの基底クラスとなっている (See: [here](no7882AgC.html) for details).

なお, "Abstract" というクラス名だが abstract class ではない 
(というか AllStatic なので関係ない. 実際このクラスのメソッドは使用されている).


```
    ((cite: hotspot/src/share/vm/interpreter/abstractInterpreter.hpp))
    // This file contains the platform-independent parts
    // of the abstract interpreter and the abstract interpreter generator.
    
    // Organization of the interpreter(s). There exists two different interpreters in hotpot
    // an assembly language version (aka template interpreter) and a high level language version
    // (aka c++ interpreter). Th division of labor is as follows:
    
    // Template Interpreter          C++ Interpreter        Functionality
    //
    // templateTable*                bytecodeInterpreter*   actual interpretation of bytecodes
    //
    // templateInterpreter*          cppInterpreter*        generation of assembly code that creates
    //                                                      and manages interpreter runtime frames.
    //                                                      Also code for populating interpreter
    //                                                      frames created during deoptimization.
    //
    // For both template and c++ interpreter. There are common files for aspects of the interpreter
    // that are generic to both interpreters. This is the layout:
    //
    // abstractInterpreter.hpp: generic description of the interpreter.
    // interpreter*:            generic frame creation and handling.
    //
    
    //------------------------------------------------------------------------------------------------------------------------
    // The C++ interface to the bytecode interpreter(s).
    
    class AbstractInterpreter: AllStatic {
```



### 詳細(Details)
See: [here](../doxygen/classAbstractInterpreter.html) for details

---
## <a name="noaChWlR_l" id="noaChWlR_l">AbstractInterpreterGenerator</a>

### 概要(Summary)
Interpreter の構築処理で使用される一時オブジェクト(StackObjクラス)の基底クラス.

実際のマシン語生成処理を行うクラス.
このクラスのサブクラスによって, 各 Interpreter クラス用のマシン語コード片が生成される (See: [here](no7882AgC.html) for details).


```
    ((cite: hotspot/src/share/vm/interpreter/abstractInterpreter.hpp))
    //------------------------------------------------------------------------------------------------------------------------
    // The interpreter generator.
    
    class Template;
    class AbstractInterpreterGenerator: public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classAbstractInterpreterGenerator.html) for details

---
