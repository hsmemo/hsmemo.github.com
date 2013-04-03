---
layout: default
title: BytecodeTracer クラス関連のクラス (BytecodeTracer, BytecodeClosure, 及びそれらの補助クラス(BytecodePrinter))
---
[Top](../index.html)

#### BytecodeTracer クラス関連のクラス (BytecodeTracer, BytecodeClosure, 及びそれらの補助クラス(BytecodePrinter))

これらは, デバッグ用(開発時用)のクラス.

### 概要(Summary)
これらは, デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

実行された全 bytecode のトレースを出力するために使用される 
(ただし, このクラスは (デバッグ時であることに加えて) TraceBytecodes オプションが指定されている場合にしか使用されない).


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.hpp))
    // class BytecodeTracer is only used by TraceBytecodes option
```

(また, TraceBytecodesAt というオプションも用意されている.
これが指定された時には, bytecode の実行回数が TraceBytecodesAt に達してからでないと出力を開始しない)


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.cpp))
      if (TraceBytecodes && BytecodeCounter::counter_value() >= TraceBytecodesAt) {
```

### 備考(Notes)
なお, このトレース処理は, BytecodeClosure のサブクラスを作って trace() メソッドをオーバーライドすることでカスタマイズできる.

(BytecodeTracer::set_closure() で登録した BytecodeClosure オブジェクトの trace() メソッドが呼ばれる)


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.hpp))
    // The BytecodeTracer is a helper class used by the interpreter for run-time
    // bytecode tracing. If bytecode tracing is turned on, trace() will be called
    // for each bytecode.
    //
    // By specialising the BytecodeClosure, all kinds of bytecode traces can
    // be done.
```

(後述のように) 現在のコードでは, デフォルトでは BytecodeTracer::std_closure() の返値が set_closure() されているため, 
カスタマイズ時には BytecodeTracer::std_closure() を上書きすればよい模様.

そして BytecodeTracer::std_closure() は, 
デフォルトでは BytecodePrinter というクラスのインスタンスを返すようになっている.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.cpp))
    static BytecodePrinter std_closure;
    BytecodeClosure* BytecodeTracer::std_closure() {
      return &::std_closure;
    }
```

とはいえ, このような set_closure() の仕組みは汎用的すぎる (そして誰も使っていないだろう), とのこと.
デバッグ用機能だから後回しになっているけど, クリーンナップされるかもしれない.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.cpp))
    // %%% This set_closure thing seems overly general, given that
    // nobody uses it.  Also, if BytecodePrinter weren't hidden
    // then methodOop could use instances of it directly and it
    // would be easier to remove races on _current_method and bcp.
    // Since this is not product functionality, we can defer cleanup.
```



### クラス一覧(class list)

  * [BytecodeTracer](#nojLIOxA1m)
  * [BytecodeClosure](#noh3RbPB5S)
  * [BytecodePrinter](#noN8ey_orn)


---
## <a name="nojLIOxA1m" id="nojLIOxA1m">BytecodeTracer</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

実行された全 bytecode のトレース出力を行うためのクラス (より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

(なお, このクラスは (デバッグ時であることに加えて) TraceBytecodes オプションが指定されている場合にしか使用されない)


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.hpp))
    #ifndef PRODUCT
```


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.hpp))
    class BytecodeTracer: AllStatic {
```

### 使われ方(Usage)
#### このクラスの初期化箇所(where this class is initialized)
interpreter_init() 内で初期化されている (といっても単に set_closure() が呼ばれているだけだが...).

現在のコードでは, デフォルトでは BytecodeTracer::std_closure() の返値が set_closure() される.


```
    ((cite: hotspot/src/share/vm/interpreter/interpreter.cpp))
    void interpreter_init() {
    ...
    #ifndef PRODUCT
      if (TraceBytecodes) BytecodeTracer::set_closure(BytecodeTracer::std_closure());
    #endif // PRODUCT
```

#### このクラスの使用箇所(where this class is used)
TemplateInterpreter の場合は, 
TemplateInterpreterGenerator::generate_and_dispatch() が生成するディスパッチコード内から 
(TemplateInterpreterGenerator::trace_bytecode() 経由で) トレース処理用のコードが呼び出される.


```
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.cpp))
    void TemplateInterpreterGenerator::generate_and_dispatch(Template* t, TosState tos_out) {
    ...
      if (TraceBytecodes)                                            trace_bytecode(t);
```


なお, 呼び出されるトレース処理用のコードは, TemplateInterpreterGenerator::generate_all() での初期化時に, 
TemplateInterpreterGenerator::generate_trace_code() によって生成されている.

(生成されたコード列から (SharedRuntime::trace_bytecode() 経由で) BytecodeTracer::trace() が呼び出され, トレース処理が行われる)


```
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.cpp))
    void TemplateInterpreterGenerator::generate_all() {
    ...
    #ifndef PRODUCT
      if (TraceBytecodes) {
        CodeletMark cm(_masm, "bytecode tracing support");
        Interpreter::_trace_code =
          EntryPoint(
            generate_trace_code(btos),
            generate_trace_code(ctos),
            generate_trace_code(stos),
            generate_trace_code(atos),
            generate_trace_code(itos),
            generate_trace_code(ltos),
            generate_trace_code(ftos),
            generate_trace_code(dtos),
            generate_trace_code(vtos)
          );
      }
```

CPP interpreter の場合には, 以下のマクロから SharedRuntime::trace_bytecode() 経由で呼び出されている.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeInterpreter.cpp))
    #ifdef PRODUCT
    #define DO_UPDATE_INSTRUCTION_COUNT(opcode)
    #else
    #define DO_UPDATE_INSTRUCTION_COUNT(opcode)                                                          \
    {                                                                                                    \
    ...
        if (TraceBytecodes) {                                                                            \
          CALL_VM((void)SharedRuntime::trace_bytecode(THREAD, 0,               \
                                       topOfStack[Interpreter::expr_index_at(1)],   \
                                       topOfStack[Interpreter::expr_index_at(2)]),  \
                                       handle_exception);                      \
        }                                                                      \
    }
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classBytecodeTracer.html) for details

---
## <a name="noh3RbPB5S" id="noh3RbPB5S">BytecodeClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

BytecodeTracer 機能において, 実際のトレース出力処理を担当するクラスの基底クラス.
なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.hpp))
    #ifndef PRODUCT
```


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.hpp))
    // For each bytecode, a BytecodeClosure's trace() routine will be called.
    
    class BytecodeClosure {
```

### 使われ方(Usage)
使用する際には, trace() メソッドをオーバーライドしたサブクラスを作ればいい.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.hpp))
      virtual void trace(methodHandle method, address bcp, uintptr_t tos, uintptr_t tos2, outputStream* st) = 0;
      virtual void trace(methodHandle method, address bcp, outputStream* st) = 0;
```

### 備考(Notes)
デフォルトでは BytecodePrinter というサブクラスが用意されている.




### 詳細(Details)
See: [here](../doxygen/classBytecodeClosure.html) for details

---
## <a name="noN8ey_orn" id="noN8ey_orn">BytecodePrinter</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

デフォルトで用意されている唯一の BytecodeClosure のサブクラス.

デフォルトでは, TraceBytecodes 指定時にはこのクラスが使用される.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.hpp))
    #ifndef PRODUCT
```


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.cpp))
    // Standard closure for BytecodeTracer: prints the current bytecode
    // and its attributes using bytecode-specific information.
    
    class BytecodePrinter: public BytecodeClosure {
```




### 詳細(Details)
See: [here](../doxygen/classBytecodePrinter.html) for details

---
