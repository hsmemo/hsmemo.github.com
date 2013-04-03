---
layout: default
title: AbstractCompiler クラス 
---
[Top](../index.html)

#### AbstractCompiler クラス 



---
## <a name="noF6Jn__xO" id="noF6Jn__xO">AbstractCompiler</a>

### 概要(Summary)
JITコンパイル処理を行うクラス(Compilerクラス)の基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのは以下のサブクラス (See: [here](no7882MiN.html) for details).

  * Compiler (C1 用)
  * C2Compiler (C2 用)
  * SharkCompiler (Shark 用)


```
    ((cite: hotspot/src/share/vm/compiler/abstractCompiler.hpp))
    class AbstractCompiler : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classAbstractCompiler.html) for details

---
