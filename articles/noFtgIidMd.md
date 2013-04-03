---
layout: default
title: InterfaceSupport クラス関連のクラス (HandleMarkCleaner, InterfaceSupport, ThreadStateTransition, ThreadInVMfromJava, ThreadInVMfromUnknown, ThreadInVMfromNative, ThreadToNativeFromVM, ThreadBlockInVM, ThreadInVMfromJavaNoAsyncException, VMEntryWrapper, VMNativeEntryWrapper, RuntimeHistogramElement)
---
[Top](../index.html)

#### InterfaceSupport クラス関連のクラス (HandleMarkCleaner, InterfaceSupport, ThreadStateTransition, ThreadInVMfromJava, ThreadInVMfromUnknown, ThreadInVMfromNative, ThreadToNativeFromVM, ThreadBlockInVM, ThreadInVMfromJavaNoAsyncException, VMEntryWrapper, VMNativeEntryWrapper, RuntimeHistogramElement)

これらは, HotSpot 内でのスレッド制御用のクラス.
より具体的に言うと, スレッドの状態遷移に関する処理を行うクラス (これらは Safepoint 停止処理等に使用される).


### クラス一覧(class list)

  * [HandleMarkCleaner](#noCdHwFqFx)
  * [InterfaceSupport](#no1PW3Ib4o)
  * [ThreadStateTransition](#noumPwyesi)
  * [ThreadInVMfromJava](#nolaeewDXF)
  * [ThreadInVMfromUnknown](#noUojUwXYF)
  * [ThreadInVMfromNative](#noAA4p3cyA)
  * [ThreadToNativeFromVM](#nofVyKeyqC)
  * [ThreadBlockInVM](#noG29HW5ak)
  * [ThreadInVMfromJavaNoAsyncException](#nox2I_x_lh)
  * [VMEntryWrapper](#notyEPToo2)
  * [VMNativeEntryWrapper](#noC-6XWV_g)
  * [RuntimeHistogramElement](#nonl8GJj-Y)


---
## <a name="noCdHwFqFx" id="noCdHwFqFx">HandleMarkCleaner</a>

### 概要(Summary)
特殊な HandleMark クラス (といっても正確には HandleMark のサブクラスじゃないが...).

VM 内の処理(= ランタイムの処理)に遷移する場面でのみ使用される補助クラス(StackObjクラス).
VM 内での処理で確保した Handle をまとめて解放する.

(なお, HandleMark より用途が限定されているので, それに合わせた軽量化も行われている, とのこと)


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // Wrapper for all entry points to the virtual machine.
    // The HandleMarkCleaner is a faster version of HandleMark.
    // It relies on the fact that there is a HandleMark further
    // down the stack (in JavaCalls::call_helper), and just resets
    // to the saved values in that HandleMark.
    
    class HandleMarkCleaner: public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* VM 内へのエントリポイントとなる各関数の先頭

  __ENTRY() マクロ

* VM 内へのエントリポイントとなっている各ブロックの先頭

  JRT_BLOCK_ENTRY() マクロ

* JIT コンパイル中にランタイムの処理が必要になった場合

   VM_ENTRY_MARK マクロ

* C++ Interpreter による実行中に例外処理を行う場合

  BytecodeInterpreter::run()

  BytecodeInterpreter::runWithChecks()

* C++ Interpreter による実行中に Safepoint 停止する場合

  SAFEPOINT マクロ

### 内部構造(Internal structure)
コンストラクタで HandleMark::push() を呼んで最後の HandleMark が記録した値を取得し, 
デストラクタで HandleMark::pop_and_restore() を呼んでその値まで HandleArea を戻すだけ.


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
      HandleMarkCleaner(Thread* thread) {
        _thread = thread;
        _thread->last_handle_mark()->push();
      }
      ~HandleMarkCleaner() {
        _thread->last_handle_mark()->pop_and_restore();
      }
```

#### 補足
HandleMarkCleaner の挙動については, JavaCalls::call_helper() と合わせて把握する必要がある.

Java コードから VM 内部に戻る際には, 
必ずスタック上のそれ以前のフレームとして
VM 内部から Java コードを呼び出した JavaCalls::call_helper() が存在している.

この JavaCalls::call_helper() 内では, Java コードを呼び出す直前に1つ HandleMark が置かれている.
HandleMarkCleaner は単に, この HandleMark が記録した値まで HandleArea を戻すだけ.

なお, 直近の HandleMark が記録した値まで戻していいのは, 以下の理由による.

* JavaCalls::call_helper() 内では, この HandleMark の作成後に Handle は作成されない.
* その後の Java コード内でも Handle は作成されない. 

(以上の理由から, VM 内への遷移時にも HandleArea の値は直近の HandleMark が記録した位置のままになっている)

### 備考(Notes)
ちなみに, JavaCalls::call_helper() 内に置かれている HandleMark とはこれのこと
(本当に呼び出し直前に置かれている).


```
    ((cite: hotspot/src/share/vm/runtime/javaCalls.cpp))
    void JavaCalls::call_helper(JavaValue* result, methodHandle* m, JavaCallArguments* args, TRAPS) {
    ...
      // do call
      { JavaCallWrapper link(method, receiver, result, CHECK);
        { HandleMark hm(thread);  // HandleMark used by HandleMarkCleaner
    
          StubRoutines::call_stub()(
            (address)&link,
            // (intptr_t*)&(result->_value), // see NOTE above (compiler problem)
            result_val_address,          // see NOTE above (compiler problem)
            result_type,
            method(),
            entry_point,
            args->parameters(),
            args->size_of_parameters(),
            CHECK
          );
```




### 詳細(Details)
See: [here](../doxygen/classHandleMarkCleaner.html) for details

---
## <a name="no1PW3Ib4o" id="no1PW3Ib4o">InterfaceSupport</a>

### 概要(Summary)
VM 内の処理(= ランタイムの処理)に遷移する際, 
及びランタイム内から戻ってくる際の処理を納めた名前空間(AllStatic クラス).

(より具体的に言うと, 主に __LEAF マクロおよび
 __ENTRY マクロ (これらは VM 内へのエントリポイントを示すマクロ) 内で使用されるクラス).


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // InterfaceSupport provides functionality used by the __LEAF and __ENTRY
    // macros. These macros are used to guard entry points into the VM and
    // perform checks upon leave of the VM.
    
    
    class InterfaceSupport: AllStatic {
```

ただし, ほとんどのフィールド／メソッドが #ifdef ASSERT 時にしか定義されないので, 
実質上はデバッグ用(開発時用)のクラスに近い
(現状では, #ifdef ASSERT 時以外でも定義されるのは os/ 下で定義されている InterfaceSupport::serialize_memory() のみ).

### 使われ方(Usage)
以下の箇所で(のみ)使用されている
(ただし, InterfaceSupport::serialize_memory() 以外はデバッグ時にしか使用されない).

* InterfaceSupport::serialize_memory() の使用箇所

    * transition_and_fence()
    * transition_from_native()
    * JavaThread::check_safepoint_and_suspend_for_native_trans()

* それ以外の使用箇所 (= デバッグ用のコード (#ifdef ASSERT 等) からの使用箇所)

    * VMEntryWrapper::VMEntryWrapper()
    * VMEntryWrapper::~VMEntryWrapper()
    * VMNativeEntryWrapper::VMNativeEntryWrapper()
    * VMNativeEntryWrapper::~VMNativeEntryWrapper()
    * TRACE_CALL() マクロ
    * Thread::check_for_valid_safepoint_state()
    * VMThread::loop()

### 内部構造(Internal structure)
share/ 部で定義されている内容 ("// OS dependent stuff" より前の内容) は全て #ifdef ASSERT 時にしか定義されない.


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    # ifdef ASSERT
     public:
      static long _scavenge_alot_counter;
      static long _fullgc_alot_counter;
      static long _number_of_calls;
      static long _fullgc_alot_invocation;
    
      // tracing
      static void trace(const char* result_type, const char* header);
    
      // Helper methods used to implement +ScavengeALot and +FullGCALot
      static void check_gc_alot() { if (ScavengeALot || FullGCALot) gc_alot(); }
      static void gc_alot();
    
      static void walk_stack_from(vframe* start_vf);
      static void walk_stack();
    
    # ifdef ENABLE_ZAP_DEAD_LOCALS
      static void zap_dead_locals_old();
    # endif
    
      static void zombieAll();
      static void unlinkSymbols();
      static void deoptimizeAll();
      static void stress_derived_pointers();
      static void verify_stack();
      static void verify_last_frame();
    # endif
    
     public:
      // OS dependent stuff
    #ifdef TARGET_OS_FAMILY_linux
    # include "interfaceSupport_linux.hpp"
    #endif
    #ifdef TARGET_OS_FAMILY_solaris
    # include "interfaceSupport_solaris.hpp"
    #endif
    #ifdef TARGET_OS_FAMILY_windows
    # include "interfaceSupport_windows.hpp"
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classInterfaceSupport.html) for details

---
## <a name="noumPwyesi" id="noumPwyesi">ThreadStateTransition</a>

### 概要(Summary)
Safepoint 停止処理で使用される一時オブジェクト(StackObjクラス) の基底クラス.

スレッドの状態遷移処理をソースコード上のスコープに合わせて行ってくれる 
(まずコンストラクタで何らかの状態に遷移させ, 次にデストラクタで何らかの状態に遷移させる) (See: [here](noadKcOM5n.html) for details).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // Basic class for all thread transition classes.
    
    class ThreadStateTransition : public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classThreadStateTransition.html) for details

---
## <a name="nolaeewDXF" id="nolaeewDXF">ThreadInVMfromJava</a>

### 概要(Summary)
ThreadStateTransition クラスの具象サブクラスの1つ.

このクラスは, Java プログラムを実行している状態 (_thread_in_Java) からランタイム内 (_thread_in_vm) への遷移時用 (See: [here](noadKcOM5n.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    class ThreadInVMfromJava : public ThreadStateTransition {
```




### 詳細(Details)
See: [here](../doxygen/classThreadInVMfromJava.html) for details

---
## <a name="noUojUwXYF" id="noUojUwXYF">ThreadInVMfromUnknown</a>

### 概要(Summary)
ThreadStateTransition クラスに類似したクラス.

このクラスは, 不定な状態からランタイム内 (_thread_in_vm) への遷移時用 (See: [here](noadKcOM5n.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    class ThreadInVMfromUnknown {
```

### 内部構造(Internal structure)
現在は, ネイティブコードを実行している状態以外で実行しても何も起こらない. 
このため実質上は「ネイティブコードを実行している状態からランタイム内への遷移」を行うクラス.
なお ThreadInVMfromNative との違いは, ネイティブコード実行中以外でも呼び出して問題ない(おかしなことにはならない)という点
(See: [here](noadKcOM5n.html) for details).




### 詳細(Details)
See: [here](../doxygen/classThreadInVMfromUnknown.html) for details

---
## <a name="noAA4p3cyA" id="noAA4p3cyA">ThreadInVMfromNative</a>

### 概要(Summary)
ThreadStateTransition クラスの具象サブクラスの1つ.

このクラスは, ネイティブコードを実行している状態 (_thread_in_native) からランタイム内 (_thread_in_vm) への遷移時用 (See: [here](noadKcOM5n.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    class ThreadInVMfromNative : public ThreadStateTransition {
```




### 詳細(Details)
See: [here](../doxygen/classThreadInVMfromNative.html) for details

---
## <a name="nofVyKeyqC" id="nofVyKeyqC">ThreadToNativeFromVM</a>

### 概要(Summary)
ThreadStateTransition クラスの具象サブクラスの1つ.

このクラスは, ランタイム内 (_thread_in_vm) からネイティブコード実行状態 (_thread_in_native) への遷移時用 (See: [here](noadKcOM5n.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    class ThreadToNativeFromVM : public ThreadStateTransition {
```




### 詳細(Details)
See: [here](../doxygen/classThreadToNativeFromVM.html) for details

---
## <a name="noG29HW5ak" id="noG29HW5ak">ThreadBlockInVM</a>

### 概要(Summary)
ThreadStateTransition クラスの具象サブクラスの1つ.

このクラスは, ランタイム内 (_thread_in_vm) からブロックされている状態/待機状態 (_thread_blocked) への遷移時用 (See: [here](noadKcOM5n.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    class ThreadBlockInVM : public ThreadStateTransition {
```




### 詳細(Details)
See: [here](../doxygen/classThreadBlockInVM.html) for details

---
## <a name="nox2I_x_lh" id="nox2I_x_lh">ThreadInVMfromJavaNoAsyncException</a>

### 概要(Summary)
ThreadStateTransition クラスの具象サブクラスの1つ.

このクラスは, Java プログラムを実行している状態 (_thread_in_java) からランタイム内 (_thread_in_vm) への遷移時用.
このため, 役割としては ThreadInVMfromJava クラスとほぼ同じ.
違いは (名前の通りだが) AsyncException が決して発生しないケース用に最適化されている点.

(より具体的に言うと, これらのクラスのデストラクタが異常を見つけると 
 JavaThread::handle_special_runtime_exit_condition() を呼び出すが,
 その際の引数が false になっており asynchrounous exception 用の処理がスキップされる, という点) (See: [here](noadKcOM5n.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // This special transition class is only used to prevent asynchronous exceptions
    // from being installed on vm exit in situations where we can't tolerate them.
    // See bugs: 4324348, 4854693, 4998314, 5040492, 5050705.
    class ThreadInVMfromJavaNoAsyncException : public ThreadStateTransition {
```




### 詳細(Details)
See: [here](../doxygen/classThreadInVMfromJavaNoAsyncException.html) for details

---
## <a name="notyEPToo2" id="notyEPToo2">VMEntryWrapper</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

VM 内(ランタイム内)に入る際と出る際のチェック処理を行う一時オブジェクト.


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // Debug class instantiated in JRT_ENTRY and ITR_ENTRY macro.
    // Can be used to verify properties on enter/exit of the VM.
    
    #ifdef ASSERT
    class VMEntryWrapper {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で VMEntryWrapper 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* IRT_ENTRY() マクロ
* IRT_ENTRY_NO_ASYNC() マクロ
* JRT_ENTRY() マクロ
* JRT_ENTRY_NO_ASYNC() マクロ
* JRT_BLOCK() マクロ

### 内部構造(Internal structure)
コンストラクタとデストラクタでチェックを行う.

なお, 以下のコマンドラインオプションでチェックする内容を制御可能.

  * VerifyLastFrame
  * WalkStackALot
  * ZapDeadLocalsOld
  * StressDerivedPointers
  * DeoptimizeALot, DeoptimizeRandom
  * ZombieALot
  * UnlinkSymbolsALot
  * VerifyStack


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
      VMEntryWrapper() {
        if (VerifyLastFrame) {
          InterfaceSupport::verify_last_frame();
        }
      }
    
      ~VMEntryWrapper() {
        InterfaceSupport::check_gc_alot();
        if (WalkStackALot) {
          InterfaceSupport::walk_stack();
        }
    #ifdef ENABLE_ZAP_DEAD_LOCALS
        if (ZapDeadLocalsOld) {
          InterfaceSupport::zap_dead_locals_old();
        }
    #endif
    #ifdef COMPILER2
        // This option is not used by Compiler 1
        if (StressDerivedPointers) {
          InterfaceSupport::stress_derived_pointers();
        }
    #endif
        if (DeoptimizeALot || DeoptimizeRandom) {
          InterfaceSupport::deoptimizeAll();
        }
        if (ZombieALot) {
          InterfaceSupport::zombieAll();
        }
        if (UnlinkSymbolsALot) {
          InterfaceSupport::unlinkSymbols();
        }
        // do verification AFTER potential deoptimization
        if (VerifyStack) {
          InterfaceSupport::verify_stack();
        }
    
      }
```




### 詳細(Details)
See: [here](../doxygen/classVMEntryWrapper.html) for details

---
## <a name="noC-6XWV_g" id="noC-6XWV_g">VMNativeEntryWrapper</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

VM 内(ランタイム内)に入る際と出る際のチェック処理を行う一時オブジェクト.
特にこのクラスは「ネイティブコードを実行している状態 (_thread_in_native) からランタイム内 (_thread_in_vm)」への遷移時にチェックを行う.


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    class VMNativeEntryWrapper {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で VMEntryWrapper 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* VM_ENTRY_MARK マクロ
* VM_QUICK_ENTRY_MARK マクロ
* JvmtiEnv::GetThreadLocalStorage()
* JvmtiExport::get_jvmti_interface()
* JNI_ENTRY_NO_PRESERVE() マクロ
* JNI_QUICK_ENTRY() マクロ
* JVM_ENTRY() マクロ
* JVM_ENTRY_NO_ENV() マクロ
* JVM_QUICK_ENTRY() マクロ

### 内部構造(Internal structure)
コンストラクタとデストラクタでチェックを行う.

なお, 以下のコマンドラインオプションでチェックする内容を制御可能.

  * GCALotAtAllSafepoints


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
      VMNativeEntryWrapper() {
        if (GCALotAtAllSafepoints) InterfaceSupport::check_gc_alot();
      }
    
      ~VMNativeEntryWrapper() {
        if (GCALotAtAllSafepoints) InterfaceSupport::check_gc_alot();
      }
```




### 詳細(Details)
See: [here](../doxygen/classVMNativeEntryWrapper.html) for details

---
## <a name="nonl8GJj-Y" id="nonl8GJj-Y">RuntimeHistogramElement</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない)
(なお, このクラスは (デバッグ時であることに加えて) 
 CountRuntimeCalls オプションが指定されている場合にしか働かない).

VM 内(ランタイム内)へのエントリポイントとなる各関数について, 
それぞれが何回呼び出されたかを記録する HistogramElement クラス.


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // VM-internal runtime interface support
    
    #ifdef ASSERT
    
    class RuntimeHistogramElement : public HistogramElement {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* TRACE_CALL() マクロ が展開される各関数内の e という static 変数
* RuntimeHistogram 大域変数
  
  (正確には, このフィールドは Histogram オブジェクトを格納するフィールド.
  この中に, 生成された全ての RuntimeHistogramElement オブジェクトが格納されている)

  (ただし, 格納している RuntimeHistogramElement オブジェクト自体は e static 変数が指しているものと重複).

#### 生成箇所(where its instances are created)
TRACE_CALL() マクロ 内で(のみ)生成されている.

(より正確に言うと, 関数内 static 変数である e の初期値として生成されている.
このため, その関数の初回の呼び出し時に生成処理が行われる)

#### 情報の記録箇所(where information is recorded)
TRACE_CALL() マクロ 内で(のみ)記録されている.

#### 情報の出力箇所(where the recorded information is output)
現在は, print_statistics() 内で(のみ)出力されている.


```
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    void print_statistics() {
    
    #ifdef ASSERT
    
      if (CountRuntimeCalls) {
        extern Histogram *RuntimeHistogram;
        RuntimeHistogram->print();
      }
```

#### 参考(for your information): TRACE_CALL() マクロ  (#ifdef ASSERT の場合)

```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    #define TRACE_CALL(result_type, header)                            \
      InterfaceSupport::_number_of_calls++;                            \
      if (TraceRuntimeCalls)                                           \
        InterfaceSupport::trace(#result_type, #header);                \
      if (CountRuntimeCalls) {                                         \
        static RuntimeHistogramElement* e = new RuntimeHistogramElement(#header); \
        if (e != NULL) e->increment_count();                           \
      }
```




### 詳細(Details)
See: [here](../doxygen/classRuntimeHistogramElement.html) for details

---
