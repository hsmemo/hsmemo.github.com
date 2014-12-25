---
layout: default
title: BytecodeHistogram クラス関連のクラス (BytecodeCounter, BytecodeHistogram, BytecodePairHistogram, 及びそれらの補助クラス(HistoEntry))
---
[Top](../index.html)

#### BytecodeHistogram クラス関連のクラス (BytecodeCounter, BytecodeHistogram, BytecodePairHistogram, 及びそれらの補助クラス(HistoEntry))

これらは, デバッグ用(開発時用)のクラス.

### 概要(Summary)
これらのクラスは, バイトコードが何回実行されたかを計測する.
以下の 3種類が用意されている.

  * BytecodeCounter		-- (bytecode 種別にかかわらず) 全体として何個の bytecode が実行されたか?
  * BytecodeHistogram		-- それぞれの bytecode が, 何回実行されたか?
  * BytecodePairHistogram	-- 連続した２個のbytecode実行列(2-gram)を見たときに, どのペアが何回実行されたか?


### クラス一覧(class list)

  * [BytecodeCounter](#noCTtovx9m)
  * [BytecodeHistogram](#noCOObKosR)
  * [BytecodePairHistogram](#no3weYevPF)
  * [HistoEntry](#noCASi5eQS)


---
## <a name="noCTtovx9m" id="noCTtovx9m">BytecodeCounter</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) 
(See: CountBytecodes, TraceBytecodes, StopInterpreterAt).

単に「全部で何個のバイトコードが実行されたか」を計測するだけのクラス (より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeHistogram.hpp))
    // BytecodeCounter counts the number of bytecodes executed
    
    class BytecodeCounter: AllStatic {
```

### 使われ方(Usage)
#### このクラスの初期化箇所(where this class is initialized)
初期化は AbstractInterpreter::initialize() 内で行われている.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/interpreter.cpp))
    void AbstractInterpreter::initialize() {
    ...
      if (CountBytecodes || TraceBytecodes || StopInterpreterAt) BytecodeCounter::reset();
      if (PrintBytecodeHistogram)                                BytecodeHistogram::reset();
      if (PrintBytecodePairHistogram)                            BytecodePairHistogram::reset();
```

#### 情報の記録箇所(where information is recorded)
現在は, 以下の箇所で(のみ)記録されている.

* TemplateInterpreterGenerator::generate_and_dispatch()  (TemplateInterpreter の場合)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.cpp))
    void TemplateInterpreterGenerator::generate_and_dispatch(Template* t, TosState tos_out) {
      if (PrintBytecodeHistogram)                                    histogram_bytecode(t);
    #ifndef PRODUCT
      // debugging code
      if (CountBytecodes || TraceBytecodes || StopInterpreterAt > 0) count_bytecode();
      if (PrintBytecodePairHistogram)                                histogram_bytecode_pair(t);
```

* DO_UPDATE_INSTRUCTION_COUNT() マクロ内                  (CppInterpreter の場合)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeInterpreter.cpp))
    #ifdef PRODUCT
    #define DO_UPDATE_INSTRUCTION_COUNT(opcode)
    #else
    #define DO_UPDATE_INSTRUCTION_COUNT(opcode)                                                          \
    {                                                                                                    \
        BytecodeCounter::_counter_value++;                                                               \
        BytecodeHistogram::_counters[(Bytecodes::Code)opcode]++;                                         \
        if (StopInterpreterAt && StopInterpreterAt == BytecodeCounter::_counter_value) os::breakpoint(); \
        if (TraceBytecodes) {                                                                            \
          CALL_VM((void)SharedRuntime::trace_bytecode(THREAD, 0,               \
                                       topOfStack[Interpreter::expr_index_at(1)],   \
                                       topOfStack[Interpreter::expr_index_at(2)]),  \
                                       handle_exception);                      \
        }                                                                      \
    }
    #endif
```

#### 情報の出力箇所(where the recorded information is output)
蓄えた結果は, デバッグ用(開発時用)の統計情報として print_statistics() 内で出力される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    void print_statistics() {
    ...
      if (CountBytecodes || TraceBytecodes || StopInterpreterAt) {
        BytecodeCounter::print();
      }
      if (PrintBytecodePairHistogram) {
        BytecodePairHistogram::print();
      }
```

#### このクラスの使用箇所(where this class is used)
print_statistics() での出力以外では, 以下の箇所で使用されている.

* バイトコードの実行回数が閾値に達した際に実行を停止させるために, このカウンタの値が使われていたりする (StopInterpreterAt オプション). 


```cpp
    ((cite: hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp))
    void TemplateInterpreterGenerator::stop_interpreter_at() {
      AddressLiteral counter(&BytecodeCounter::_counter_value);
      __ load_contents(counter, G3_scratch);
      AddressLiteral stop_at(&StopInterpreterAt);
      __ load_ptr_contents(stop_at, G4_scratch);
      __ cmp(G3_scratch, G4_scratch);
      __ breakpoint_trap(Assembler::equal);
```


* また, BytecodeTracer クラスの機能を実現するため (bytecode の実行回数がデバッグ用の閾値に達したかどうかを見るため) 
  にも使われていたりする (TraceBytecodesAt オプション).


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.cpp))
      if (TraceBytecodes && BytecodeCounter::counter_value() >= TraceBytecodesAt) {
```


* また, BytecodePrinter での実際の出力時に一緒に表示されている模様.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeTracer.cpp))
      void trace(methodHandle method, address bcp, uintptr_t tos, uintptr_t tos2, outputStream* st) {
    ...
        if (Verbose) {
          st->print("%8d  %4d  " INTPTR_FORMAT " " INTPTR_FORMAT " %s",
               BytecodeCounter::counter_value(), bci, tos, tos2, Bytecodes::name(code));
        } else {
          st->print("%8d  %4d  %s",
               BytecodeCounter::counter_value(), bci, Bytecodes::name(code));
        }
```




### 詳細(Details)
See: [here](../doxygen/classBytecodeCounter.html) for details

---
## <a name="noCOObKosR" id="noCOObKosR">BytecodeHistogram</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) 
(See: PrintBytecodeHistogram)

単に「各バイトコードが何回実行されたか」を計測するだけのクラス (より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeHistogram.hpp))
    // BytecodeHistogram collects number of executions of bytecodes
    
    class BytecodeHistogram: AllStatic {
```

### 使われ方(Usage)
#### このクラスの初期化箇所(where this class is initialized)
初期化は AbstractInterpreter::initialize() 内で行われている.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/interpreter.cpp))
    void AbstractInterpreter::initialize() {
    ...
      if (CountBytecodes || TraceBytecodes || StopInterpreterAt) BytecodeCounter::reset();
      if (PrintBytecodeHistogram)                                BytecodeHistogram::reset();
      if (PrintBytecodePairHistogram)                            BytecodePairHistogram::reset();
```

#### 情報の記録箇所(where information is recorded)
現在は, 以下の箇所で(のみ)記録されている.

* TemplateInterpreterGenerator::generate_and_dispatch()  (TemplateInterpreter の場合)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.cpp))
    void TemplateInterpreterGenerator::generate_and_dispatch(Template* t, TosState tos_out) {
      if (PrintBytecodeHistogram)                                    histogram_bytecode(t);
    #ifndef PRODUCT
      // debugging code
      if (CountBytecodes || TraceBytecodes || StopInterpreterAt > 0) count_bytecode();
      if (PrintBytecodePairHistogram)                                histogram_bytecode_pair(t);
```

* DO_UPDATE_INSTRUCTION_COUNT() マクロ内                  (CppInterpreter の場合)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeInterpreter.cpp))
    #ifdef PRODUCT
    #define DO_UPDATE_INSTRUCTION_COUNT(opcode)
    #else
    #define DO_UPDATE_INSTRUCTION_COUNT(opcode)                                                          \
    {                                                                                                    \
        BytecodeCounter::_counter_value++;                                                               \
        BytecodeHistogram::_counters[(Bytecodes::Code)opcode]++;                                         \
        if (StopInterpreterAt && StopInterpreterAt == BytecodeCounter::_counter_value) os::breakpoint(); \
        if (TraceBytecodes) {                                                                            \
          CALL_VM((void)SharedRuntime::trace_bytecode(THREAD, 0,               \
                                       topOfStack[Interpreter::expr_index_at(1)],   \
                                       topOfStack[Interpreter::expr_index_at(2)]),  \
                                       handle_exception);                      \
        }                                                                      \
    }
    #endif
```

#### 情報の出力箇所(where the recorded information is output)
蓄えた結果は before_exit() 内で出力されている模様.


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    void before_exit(JavaThread * thread) {
    ...
      if (PrintBytecodeHistogram) {
        BytecodeHistogram::print();
      }
```

### 内部構造(Internal structure)
内部にはバイトコード種別分だけの大きさの配列を持っており, 各バイトコードが実行されるたびに該当するエントリをincrementしている.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeHistogram.hpp))
      NOT_PRODUCT(static int _counters[Bytecodes::number_of_codes];)   // a counter for each bytecode
```




### 詳細(Details)
See: [here](../doxygen/classBytecodeHistogram.html) for details

---
## <a name="no3weYevPF" id="no3weYevPF">BytecodePairHistogram</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) 
(See: PrintBytecodePairHistogram)

2個の連続したバイトコード(2-gram)単位で見て, 「それぞれのペアが何回実行されたか」を計測するだけのクラス 
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeHistogram.hpp))
    // BytecodePairHistogram collects number of executions of bytecode pairs.
    // A bytecode pair is any sequence of two consequtive bytecodes.
    
    class BytecodePairHistogram: AllStatic {
```

### 使われ方(Usage)
#### このクラスの初期化箇所(where this class is initialized)
初期化は AbstractInterpreter::initialize() 内で行われている.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/interpreter.cpp))
    void AbstractInterpreter::initialize() {
    ...
      if (CountBytecodes || TraceBytecodes || StopInterpreterAt) BytecodeCounter::reset();
      if (PrintBytecodeHistogram)                                BytecodeHistogram::reset();
      if (PrintBytecodePairHistogram)                            BytecodePairHistogram::reset();
```

#### 情報の記録箇所(where information is recorded)
現在は, 以下の箇所で(のみ)記録されている.

* TemplateInterpreterGenerator::generate_and_dispatch()  (TemplateInterpreter の場合)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.cpp))
    void TemplateInterpreterGenerator::generate_and_dispatch(Template* t, TosState tos_out) {
      if (PrintBytecodeHistogram)                                    histogram_bytecode(t);
    #ifndef PRODUCT
      // debugging code
      if (CountBytecodes || TraceBytecodes || StopInterpreterAt > 0) count_bytecode();
      if (PrintBytecodePairHistogram)                                histogram_bytecode_pair(t);
```

* DO_UPDATE_INSTRUCTION_COUNT() マクロ内                  (CppInterpreter の場合)


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeInterpreter.cpp))
    #ifdef PRODUCT
    #define DO_UPDATE_INSTRUCTION_COUNT(opcode)
    #else
    #define DO_UPDATE_INSTRUCTION_COUNT(opcode)                                                          \
    {                                                                                                    \
        BytecodeCounter::_counter_value++;                                                               \
        BytecodeHistogram::_counters[(Bytecodes::Code)opcode]++;                                         \
        if (StopInterpreterAt && StopInterpreterAt == BytecodeCounter::_counter_value) os::breakpoint(); \
        if (TraceBytecodes) {                                                                            \
          CALL_VM((void)SharedRuntime::trace_bytecode(THREAD, 0,               \
                                       topOfStack[Interpreter::expr_index_at(1)],   \
                                       topOfStack[Interpreter::expr_index_at(2)]),  \
                                       handle_exception);                      \
        }                                                                      \
    }
    #endif
```

#### 情報の出力箇所(where the recorded information is output)
蓄えた結果は, デバッグ用(開発時用)の統計情報として print_statistics() 内で出力される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    void print_statistics() {
    ...
      if (CountBytecodes || TraceBytecodes || StopInterpreterAt) {
        BytecodeCounter::print();
      }
      if (PrintBytecodePairHistogram) {
        BytecodePairHistogram::print();
      }
```

### 内部構造(Internal structure)
内部には「バイトコード種別数の２乗」の大きさの配列を持っており, 各バイトコードが実行されるたびに該当するエントリをincrementしている.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeHistogram.hpp))
    ...
        number_of_pairs      = number_of_codes * number_of_codes
    ...
      NOT_PRODUCT(static int  _counters[number_of_pairs];)  // a counter for each pair
```




### 詳細(Details)
See: [here](../doxygen/classBytecodePairHistogram.html) for details

---
## <a name="noCASi5eQS" id="noCASi5eQS">HistoEntry</a>

### 概要(Summary)
BytecodeHistogram クラス及び BytecodePairHistogram クラス用の補助クラス.

BytecodeHistogram クラスや BytecodePairHistogram クラスの中身をソートして出力する処理で使用される一時オブジェクト(ResourceObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/interpreter/bytecodeHistogram.cpp))
    // Helper class for sorting
    
    class HistoEntry: public ResourceObj {
```




### 詳細(Details)
See: [here](../doxygen/classHistoEntry.html) for details

---
