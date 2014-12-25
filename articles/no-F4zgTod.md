---
layout: default
title: C2Compiler クラス 
---
[Top](../index.html)

#### C2Compiler クラス 



---
## <a name="noFhdYblb1" id="noFhdYblb1">C2Compiler</a>

### 概要(Summary)
AbstractCompiler クラスの具象サブクラスの1つ.

このクラスは C2 の JIT コンパイル処理用
(このクラスの C2Compiler::compile_method() メソッドが呼び出されることで
 C2 JIT コンパイル処理が開始される).


```cpp
    ((cite: hotspot/src/share/vm/opto/c2compiler.hpp))
    class C2Compiler : public AbstractCompiler {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
CompileBroker クラスの _compilers フィールド (static フィールド) に(のみ)格納されている.

(正確には, このフィールドは AbstractCompiler の配列を格納するフィールド.
この中に, 全ての AbstractCompiler オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
CompileBroker::compilation_init() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
以下の処理で使用される. (#TODO 他の使用箇所)

```
* JIT コンパイルの開始処理

  CompileBroker::compiler_thread_loop()
  -> CompileBroker::invoke_compiler_on_method()
     -> C2Compiler::compile_method()
```




### 詳細(Details)
See: [here](../doxygen/classC2Compiler.html) for details

---
