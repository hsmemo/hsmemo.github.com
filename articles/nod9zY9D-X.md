---
layout: default
title: CodeBlob とそのサブクラス (CodeBlob, BufferBlob, AdapterBlob, MethodHandlesAdapterBlob, RuntimeStub, SingletonBlob, RicochetBlob, DeoptimizationBlob, UncommonTrapBlob, ExceptionBlob, SafepointBlob)
---
[Top](../index.html)

#### CodeBlob とそのサブクラス (CodeBlob, BufferBlob, AdapterBlob, MethodHandlesAdapterBlob, RuntimeStub, SingletonBlob, RicochetBlob, DeoptimizationBlob, UncommonTrapBlob, ExceptionBlob, SafepointBlob)

これらは, 実行時に生成されたマシン語コードを格納するためのクラス (See: [here](no7882z5r.html) for details).


### クラス一覧(class list)

  * [CodeBlob](#no8SLNCP8d)
  * [BufferBlob](#noDhUbq4w_)
  * [AdapterBlob](#nompKarBYX)
  * [MethodHandlesAdapterBlob](#no-ysamnWn)
  * [RuntimeStub](#nouJQCWekS)
  * [SingletonBlob](#noVpf_3w7O)
  * [RicochetBlob](#noza5wKAcI)
  * [DeoptimizationBlob](#nobm3L7Dh5)
  * [UncommonTrapBlob](#noJeXzaNfI)
  * [ExceptionBlob](#noJ4630yq9)
  * [SafepointBlob](#nog1hEruEb)


---
## <a name="no8SLNCP8d" id="no8SLNCP8d">CodeBlob</a>

### 概要(Summary)
動的に生成されたマシン語コードを格納するメモリ領域を表すクラスの基底クラス.


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    class CodeBlob VALUE_OBJ_CLASS_SPEC {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.
(例えば, このクラスの is_alive() 等が abstract)


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
      virtual bool is_alive() const                  = 0;
```

### 内部構造(Internal structure)
CodeBlob は以下のような構造を持つ.

("relocation" の箇所に relocInfo の配列が詰まっている模様.
 relocation_begin(), relocation_end() でアクセスできる).


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    // Layout:
    //   - header
    //   - relocation
    //   - content space
    //     - instruction space
    //   - data space
```

なお, hpp ファイルで宣言されている以下のフィールド群は, 上の Layout 中での "header" に相当する模様.
そして, これらのフィールドが終わった後ろの部分(フィールドが宣言されていない部分)に, 
上記の relocation 情報や content space が存在している.

(ちなみに, oopmap 情報(OopMapSet) や block_comment 情報(CodeComments) は header 部分に含まれている).


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
      const char* _name;
      int        _size;                              // total size of CodeBlob in bytes
      int        _header_size;                       // size of header (depends on subclass)
      int        _relocation_size;                   // size of relocation
      int        _content_offset;                    // offset to where content region begins (this includes consts, insts, stubs)
      int        _code_offset;                       // offset to where instructions region begins (this includes insts, stubs)
      int        _frame_complete_offset;             // instruction offsets in [0.._frame_complete_offset) have
                                                     // not finished setting up their frame. Beware of pc's in
                                                     // that range. There is a similar range(s) on returns
                                                     // which we don't detect.
      int        _data_offset;                       // offset to where data region begins
      int        _frame_size;                        // size of stack frame
      OopMapSet* _oop_maps;                          // OopMap for this CodeBlob
      CodeComments _comments;
```



### 詳細(Details)
See: [here](../doxygen/classCodeBlob.html) for details

---
## <a name="noDhUbq4w_" id="noDhUbq4w_">BufferBlob</a>

### 概要(Summary)
relocate 処理が不要なコードを格納するための CodeBlob
(例えば, interpreter や stubroutine 用に生成されるコードが該当する).


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // BufferBlob: used to hold non-relocatable machine code such as the interpreter, stubroutines, etc.
    
    class BufferBlob: public CodeBlob {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
具体的には, 以下のスタブがこのクラスのインスタンスとなっている.


```cpp
    ((cite: hotspot/src/share/vm/interpreter/interpreterRuntime.hpp))
    class SignatureHandlerLibrary: public AllStatic {
    ...
      static BufferBlob*              _handler_blob; // the current buffer blob containing the generated handlers
```


```cpp
    ((cite: hotspot/src/share/vm/opto/compile.hpp))
      class ConstantTable {
    ...
      BufferBlob*           _scratch_buffer_blob;   // For temporary code buffers.
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.hpp))
    class AdapterHandlerLibrary: public AllStatic {
    ...
      static BufferBlob* _buffer; // the temporary code buffer in CodeCache
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubRoutines.hpp))
    class StubRoutines: AllStatic {
    ...
      static BufferBlob* _code1;                               // code buffer for initial routines
      static BufferBlob* _code2;                               // code buffer for all other routines
```

(... その他, あちこちで使われている #TODO)




### 詳細(Details)
See: [here](../doxygen/classBufferBlob.html) for details

---
## <a name="nompKarBYX" id="nompKarBYX">AdapterBlob</a>

### 概要(Summary)
c2i adapters および i2c adapters のコードを格納するための CodeBlob (See: [here](no7882a7C.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // AdapterBlob: used to hold C2I/I2C adapters
    
    class AdapterBlob: public BufferBlob {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
AdapterHandlerLibrary::get_adapter() の中で生成されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    AdapterHandlerEntry* AdapterHandlerLibrary::get_adapter(methodHandle method) {
    ...
          B = AdapterBlob::create(&buffer);
```




### 詳細(Details)
See: [here](../doxygen/classAdapterBlob.html) for details

---
## <a name="no-ysamnWn" id="no-ysamnWn">MethodHandlesAdapterBlob</a>

### 概要(Summary)
MethodHandles adapters を格納するための CodeBlob. (#TODO)


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // MethodHandlesAdapterBlob: used to hold MethodHandles adapters
    
    class MethodHandlesAdapterBlob: public BufferBlob {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
具体的には, 以下のスタブがこのクラスのインスタンスとなっている.


```cpp
    ((cite: hotspot/src/share/vm/prims/methodHandles.cpp))
    MethodHandlesAdapterBlob* MethodHandles::_adapter_code = NULL;
```




### 詳細(Details)
See: [here](../doxygen/classMethodHandlesAdapterBlob.html) for details

---
## <a name="nouJQCWekS" id="nouJQCWekS">RuntimeStub</a>

### 概要(Summary)
JIT 生成コードから使用されるランタイムスタブ(Runtime を呼び出すためのスタブ)用のコードを格納する CodeBlob.


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // RuntimeStub: describes stubs used by compiled code to call a (static) C++ runtime routine
    
    class RuntimeStub: public CodeBlob {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
具体的には, 以下のスタブがこのクラスのインスタンスとなっている.

##### SharedRuntime 内:

```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    // Shared stub locations
    RuntimeStub*        SharedRuntime::_wrong_method_blob;
    RuntimeStub*        SharedRuntime::_ic_miss_blob;
    RuntimeStub*        SharedRuntime::_resolve_opt_virtual_call_blob;
    RuntimeStub*        SharedRuntime::_resolve_virtual_call_blob;
    RuntimeStub*        SharedRuntime::_resolve_static_call_blob;
```

##### OptoRuntime 内:
OptoRuntime::generate_stub() で生成される以下のスタブが該当する.


```cpp
    ((cite: hotspot/src/share/vm/opto/runtime.cpp))
    void OptoRuntime::generate(ciEnv* env) {
    ...
      // Note: tls: Means fetching the return oop out of the thread-local storage
      //
      //   variable/name                       type-function-gen              , runtime method                  ,fncy_jp, tls,save_args,retpc
      // -------------------------------------------------------------------------------------------------------------------------------
      gen(env, _new_instance_Java              , new_instance_Type            , new_instance_C                  ,    0 , true , false, false);
      gen(env, _new_array_Java                 , new_array_Type               , new_array_C                     ,    0 , true , false, false);
      gen(env, _multianewarray2_Java           , multianewarray2_Type         , multianewarray2_C               ,    0 , true , false, false);
      gen(env, _multianewarray3_Java           , multianewarray3_Type         , multianewarray3_C               ,    0 , true , false, false);
      gen(env, _multianewarray4_Java           , multianewarray4_Type         , multianewarray4_C               ,    0 , true , false, false);
      gen(env, _multianewarray5_Java           , multianewarray5_Type         , multianewarray5_C               ,    0 , true , false, false);
      gen(env, _g1_wb_pre_Java                 , g1_wb_pre_Type               , SharedRuntime::g1_wb_pre        ,    0 , false, false, false);
      gen(env, _g1_wb_post_Java                , g1_wb_post_Type              , SharedRuntime::g1_wb_post       ,    0 , false, false, false);
      gen(env, _complete_monitor_locking_Java  , complete_monitor_enter_Type  , SharedRuntime::complete_monitor_locking_C      ,    0 , false, false, false);
      gen(env, _rethrow_Java                   , rethrow_Type                 , rethrow_C                       ,    2 , true , false, true );
    
      gen(env, _slow_arraycopy_Java            , slow_arraycopy_Type          , SharedRuntime::slow_arraycopy_C ,    0 , false, false, false);
      gen(env, _register_finalizer_Java        , register_finalizer_Type      , register_finalizer              ,    0 , false, false, false);
    
    # ifdef ENABLE_ZAP_DEAD_LOCALS
      gen(env, _zap_dead_Java_locals_Java      , zap_dead_locals_Type         , zap_dead_Java_locals_C          ,    0 , false, true , false );
      gen(env, _zap_dead_native_locals_Java    , zap_dead_locals_Type         , zap_dead_native_locals_C        ,    0 , false, true , false );
    # endif
```

とはいえ, この場合は, 作った RuntimeStub から entry point だけを取り出し, それ以外は捨ててしまっている模様.
(捨ててはいない?? CodeCache には登録されている？ #TODO)

##### StubRoutines 内:

```cpp
    ((cite: hotspot/src/share/vm/runtime/stubRoutines.cpp))
    address StubRoutines::_throw_AbstractMethodError_entry          = NULL;
    address StubRoutines::_throw_IncompatibleClassChangeError_entry = NULL;
    address StubRoutines::_throw_ArithmeticException_entry          = NULL;
    address StubRoutines::_throw_NullPointerException_entry         = NULL;
    address StubRoutines::_throw_NullPointerException_at_call_entry = NULL;
    address StubRoutines::_throw_StackOverflowError_entry           = NULL;
    address StubRoutines::_throw_WrongMethodTypeException_entry     = NULL;
```

とはいえ, この場合は, 作った RuntimeStub から entry point だけを取り出し, それ以外は捨ててしまっている模様.
(捨ててはいない?? CodeCache には登録されている？ #TODO)




### 詳細(Details)
See: [here](../doxygen/classRuntimeStub.html) for details

---
## <a name="noVpf_3w7O" id="noVpf_3w7O">SingletonBlob</a>

### 概要(Summary)
VM 内に 1つしか存在しない CodeBlob の基底クラス.

(なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラスである模様)


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // Super-class for all blobs that exist in only one instance. Implements default behaviour.
    
    class SingletonBlob: public CodeBlob {
```

具体的には以下のようなサブクラスを持つ.

* RicochetBlob
* DeoptimizationBlob
* UncommonTrapBlob
* ExceptionBlob
* SafepointBlob




### 詳細(Details)
See: [here](../doxygen/classSingletonBlob.html) for details

---
## <a name="noza5wKAcI" id="noza5wKAcI">RicochetBlob</a>

### 概要(Summary)
(#Under Construction)


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // RicochetBlob
    // Holds an arbitrary argument list indefinitely while Java code executes recursively.
    
    class RicochetBlob: public SingletonBlob {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
具体的には, 以下のスタブがこのクラスのインスタンスとなっている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    RicochetBlob*       SharedRuntime::_ricochet_blob;
```




### 詳細(Details)
See: [here](../doxygen/classRicochetBlob.html) for details

---
## <a name="nobm3L7Dh5" id="nobm3L7Dh5">DeoptimizationBlob</a>

### 概要(Summary)
脱最適化処理(deopt 処理)を行うコードを格納する CodeBlob (See: [here](no7882nFJ.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // DeoptimizationBlob
    
    class DeoptimizationBlob: public SingletonBlob {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
具体的には, 以下のスタブがこのクラスのインスタンスとなっている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    DeoptimizationBlob* SharedRuntime::_deopt_blob;
```




### 詳細(Details)
See: [here](../doxygen/classDeoptimizationBlob.html) for details

---
## <a name="noJeXzaNfI" id="noJeXzaNfI">UncommonTrapBlob</a>

### 概要(Summary)
uncommon trap 処理を行うコードを格納する CodeBlob (See: [here](no78820PP.html) for details).

現在は C2 でしか使われていない (というか, #ifdef COMPILER2 でなければ定義もされない)


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // UncommonTrapBlob (currently only used by Compiler 2)
    
    #ifdef COMPILER2
    
    class UncommonTrapBlob: public SingletonBlob {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
具体的には, 以下のスタブがこのクラスのインスタンスとなっている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    #ifdef COMPILER2
    UncommonTrapBlob*   SharedRuntime::_uncommon_trap_blob;
    #endif // COMPILER2
```




### 詳細(Details)
See: [here](../doxygen/classUncommonTrapBlob.html) for details

---
## <a name="noJ4630yq9" id="noJ4630yq9">ExceptionBlob</a>

### 概要(Summary)
例外処理を行うコードを格納する CodeBlob (See: [here](no7882BaV.html) for details).

現在は C2 でしか使われていない (というか, #ifdef COMPILER2 でなければ定義もされない)


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // ExceptionBlob: used for exception unwinding in compiled code (currently only used by Compiler 2)
    
    class ExceptionBlob: public SingletonBlob {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
具体的には, 以下のスタブがこのクラスのインスタンスとなっている.


```cpp
    ((cite: hotspot/src/share/vm/opto/runtime.cpp))
    ExceptionBlob* OptoRuntime::_exception_blob;
```




### 詳細(Details)
See: [here](../doxygen/classExceptionBlob.html) for details

---
## <a name="nog1hEruEb" id="nog1hEruEb">SafepointBlob</a>

### 概要(Summary)
safepoint polling 処理でメモリアクセス違反が起きた際の処理(safepoint 処理)を行うコードを格納する CodeBlob (See: [here](no7882Okb.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
    //----------------------------------------------------------------------------------------------------
    // SafepointBlob: handles illegal_instruction exceptions during a safepoint
    
    class SafepointBlob: public SingletonBlob {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
SharedRuntime::generate_handler_blob() で作成されている (See: SharedRuntime::generate_handler_blob()).

#### インスタンスの格納場所(where its instances are stored)
具体的には, 以下のスタブがこのクラスのインスタンスとなっている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    SafepointBlob*      SharedRuntime::_polling_page_safepoint_handler_blob;
    SafepointBlob*      SharedRuntime::_polling_page_return_handler_blob;
```




### 詳細(Details)
See: [here](../doxygen/classSafepointBlob.html) for details

---
