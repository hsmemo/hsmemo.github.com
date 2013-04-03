---
layout: default
title: ciEnv クラス 
---
[Top](../index.html)

#### ciEnv クラス 



---
## <a name="nosf4Hw69F" id="nosf4Hw69F">ciEnv</a>

### 概要(Summary)
JIT コンパイル作業用の一時オブジェクト(StackObjクラス) (See: [here](no7882MiN.html) for details).

JIT コンパイラとそれ以外の部分をつなぐ broker(仲介役) 的なクラス.
JIT コンパイラが必要とするオブジェクト／メソッドを提供したり, 
JIT コンパイル中に得られたメタ情報(デバッグ情報, OopMap, 等)の格納場所を提供したりしている.


```
    ((cite: hotspot/src/share/vm/ci/ciEnv.hpp))
    // ciEnv
    //
    // This class is the top level broker for requests from the compiler
    // to the VM.
    class ciEnv : StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* CompileBroker::invoke_compiler_on_method()  (局所変数として生成)

* ciObjectFactory::initialize()  (局所変数として生成)




### 詳細(Details)
See: [here](../doxygen/classciEnv.html) for details

---
