---
layout: default
title: BytecodeInterpreter クラス(interpreterState) 
---
[Top](../index.html)

#### BytecodeInterpreter クラス(interpreterState) 



---
## <a name="no_H_V9Kef" id="no_H_V9Kef">BytecodeInterpreter</a>

### 概要(Summary)
C++ Interpreter 用のクラス (#ifdef CC_INTERP 時にしか定義されない).


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeInterpreter.hpp))
    #ifdef CC_INTERP
```

このクラスが, C++ Interpreter における実際の各bytecodeでの挙動を定めている
(Template Interpreter における TemplateTable クラスに相当). (See: [here](no7882AgC.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/abstractInterpreter.hpp))
    // Template Interpreter          C++ Interpreter        Functionality
    //
    // templateTable*                bytecodeInterpreter*   actual interpretation of bytecodes
    //
    // templateInterpreter*          cppInterpreter*        generation of assembly code that creates
    //                                                      and manages interpreter runtime frames.
    //                                                      Also code for populating interpreter
    //                                                      frames created during deoptimization.
```


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeInterpreter.hpp))
    class BytecodeInterpreter : StackObj {
```


なお, interpreterState という型も使われるが, これは BytecodeInterpreter* のこと.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeInterpreter.hpp))
    typedef class BytecodeInterpreter* interpreterState;
```

### 内部構造(Internal structure)
実際の bytecode の処理は, BytecodeInterpreter::run() または BytecodeInterpreter::runWithChecks() で定義されている模様.

(run: というラベル以下を参照. メソッドエントリ処理などが終わった後, goto run してバイトコード処理が開始されている模様)

(runWithChecks() は JVMTI 用のフックが入ったバージョン. JVMTI のチェックが有効なら runWithChecks() が使われる. 
逆に有効でなければ run() が呼ばれている模様)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeInterpreter.cpp))
    /*
     * BytecodeInterpreter::run(interpreterState istate)
     * BytecodeInterpreter::runWithChecks(interpreterState istate)
     *
     * The real deal. This is where byte codes actually get interpreted.
     * Basically it's a big while loop that iterates until we return from
     * the method passed in.
     *
     * The runWithChecks is used if JVMTI is enabled.
     *
     */
    #if defined(VM_JVMTI)
    void
    BytecodeInterpreter::runWithChecks(interpreterState istate) {
    #else
    void
    BytecodeInterpreter::run(interpreterState istate) {
    #endif
```





### 詳細(Details)
See: [here](../doxygen/classBytecodeInterpreter.html) for details

---
