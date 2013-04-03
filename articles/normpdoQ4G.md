---
layout: default
title: InterpreterGenerator クラス 
---
[Top](../index.html)

#### InterpreterGenerator クラス 



---
## <a name="nogqY1C8jL" id="nogqY1C8jL">InterpreterGenerator</a>

### 概要(Summary)
AbstractInterpreterGenerator クラスの具象サブクラス.
ビルド時の #ifdef によって 
TemplateInterpreterGenerator か CppInterpreterGenerator のどちらかのサブクラスになる.

実質的な役割としては, 使用する Interpreter 種別を隠蔽するためのラッパークラス.

実際の InterpreterGenerator クラスとしての機能は 
TemplateInterpreterGenerator クラスもしくは CppInterpreterGenerator クラスに定義されているが, 
実際の使用箇所では Interpreter 種別の違いを隠蔽したコードにするため
InterpreterGenerator クラスとして使用されている模様 (See: [here](no7882AgC.html) for details).


```
    ((cite: hotspot/src/share/vm/interpreter/interpreterGenerator.hpp))
    // This file contains the platform-independent parts
    // of the interpreter generator.
    
    
    class InterpreterGenerator: public CC_INTERP_ONLY(CppInterpreterGenerator)
                                       NOT_CC_INTERP(TemplateInterpreterGenerator) {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* TemplateInterpreter::initialize()
* CppInterpreter::initialize()

### 内部構造(Internal structure)
InterpreterGenerator の機能はほとんどが cpu 依存なので, share/ 以下に書かれていることはほとんどない
(cpu/ 以下のファイルを #include しているだけ).


```
    ((cite: hotspot/src/share/vm/interpreter/interpreterGenerator.hpp))
    InterpreterGenerator(StubQueue* _code);
    
    #ifdef TARGET_ARCH_x86
    # include "interpreterGenerator_x86.hpp"
    #endif
    #ifdef TARGET_ARCH_sparc
    # include "interpreterGenerator_sparc.hpp"
    #endif
    #ifdef TARGET_ARCH_zero
    # include "interpreterGenerator_zero.hpp"
    #endif
    #ifdef TARGET_ARCH_arm
    # include "interpreterGenerator_arm.hpp"
    #endif
    #ifdef TARGET_ARCH_ppc
    # include "interpreterGenerator_ppc.hpp"
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classInterpreterGenerator.html) for details

---
