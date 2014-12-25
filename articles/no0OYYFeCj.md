---
layout: default
title: frame クラス関連のクラス (frame, frame::CheckValueClosure, frame::CheckOopClosure, frame::ZapDeadClosure, FrameValue, FrameValues, StackFrameStream, 及びそれらの補助クラス(InterpreterFrameClosure, InterpretedArgumentOopFinder, EntryFrameOopFinder, CompiledArgumentOopFinder))
---
[Top](../index.html)

#### frame クラス関連のクラス (frame, frame::CheckValueClosure, frame::CheckOopClosure, frame::ZapDeadClosure, FrameValue, FrameValues, StackFrameStream, 及びそれらの補助クラス(InterpreterFrameClosure, InterpretedArgumentOopFinder, EntryFrameOopFinder, CompiledArgumentOopFinder))

これらは, HotSpot 内でのスレッド管理用のクラス.
より具体的に言うと, スレッドのスタックフレームを扱うためのユーティリティ・クラス.


### クラス一覧(class list)

  * [frame](#novPtRFCpK)
  * [frame::CheckValueClosure](#noFtDd5HHh)
  * [frame::CheckOopClosure](#noJe44pmYH)
  * [frame::ZapDeadClosure](#noP3Jbsts2)
  * [FrameValues](#noshhOwSiZ)
  * [FrameValue](#nove92Uhha)
  * [StackFrameStream](#noUfMWLP1E)
  * [InterpreterFrameClosure](#nopQhUUk-u)
  * [InterpretedArgumentOopFinder](#nonGOYNKud)
  * [EntryFrameOopFinder](#noC0SULiUS)
  * [CompiledArgumentOopFinder](#noxNORMtNS)


---
## <a name="novPtRFCpK" id="novPtRFCpK">frame</a>

### 概要(Summary)
スタックフレーム(activation record)を扱うためのユーティリティ・クラス.
1つの frame オブジェクトが 1つのスタックフレームに対応する.

なお, HotSpot 内で使用されるスタックフレームには幾つかの種別があるが, 
このクラスはどの種別のスタックフレームでも扱える
(例: C のフレーム, Java の (JIT コンパイルされたメソッドの) フレーム, Java の (Interpreter 実行されているメソッドの) フレーム).

なお, JIT コンパイラによってインライン展開が行われていた場合, 
スタックフレームは Java レベルのメソッドと1対1対応しないことがある
(実際のスタックフレーム 1 に対して Java レベルのメソッド n 個が対応).
Java レベルのメソッドと 1対1対応するような(論理的な)スタックフレームの情報が欲しい場合は, 
このクラスの代わりに vframe クラスを使用すればいい.


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
    // A frame represents a physical stack frame (an activation).  Frames
    // can be C or Java frames, and the Java frames can be interpreted or
    // compiled.  In contrast, vframes represent source-level activations,
    // so that one physical frame can correspond to multiple source level
    // frames because of inlining.
    
    class frame VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 RFrame オブジェクトの _fr フィールド
  
  (ただし, ResourceObjクラスのフィールドなので一時的なオブジェクト)
   
* 各 vframeArrayElement オブジェクトの _frame フィールド

  (ただし, vframeArrayElement オブジェクトのフィールドなので一時的なオブジェクト)

* 各 vframeArray オブジェクトの _original フィールド

  (ただし, vframeArray オブジェクトのフィールドなので一時的なオブジェクト)

* 各 vframeArray オブジェクトの _caller フィールド

  (ただし, vframeArray オブジェクトのフィールドなので一時的なオブジェクト)

* 各 vframeArray オブジェクトの _sender フィールド
  
  (ただし, vframeArray オブジェクトのフィールドなので一時的なオブジェクト)
  
* 各 vframe オブジェクトの _fr フィールド

  (ただし, ResourceObjクラスのフィールドなので一時的なオブジェクト)

* 各 vframeStreamCommon オブジェクトの _frame フィールド

  (ただし, StackObjクラスのフィールドなので一時的なオブジェクト)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ValueObj クラスなので「生成」というのは少し違和感があるが, 以下の箇所でのみ新しい値を持ったインスタンスが生成されている. 
他の使用箇所はコピーコンストラクタ, あるいは既に生成済みの値へのポインタ).

* (上記のフィールドは全て, ポインタ型ではなく実体なので,
  格納しているオブジェクトの生成時に一緒に生成される)

* os::fetch_frame_from_context(void* ucVoid)
* os::get_sender_for_C_frame()
* os::current_frame()

* JavaThread::pd_get_top_frame_for_signal_handler()
* JavaThread::pd_last_frame()

* frame::sender()
* frame::java_sender()
* frame::profile_find_Java_sender_frame()

* MethodHandles::ricochet_frame_sender()




### 詳細(Details)
See: [here](../doxygen/classframe.html) for details

---
## <a name="noFtDd5HHh" id="noFtDd5HHh">frame::CheckValueClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef ENABLE_ZAP_DEAD_LOCALS 時にしか定義されない).

スタックフレーム中の oop 以外の格納場所について, 
誤って oop が格納されていないかどうかをチェックする Closure クラス

(より正確に言うと,「oop の可能性が考えられる値(ヒープ内を指しているポインタと考えられる値)」が格納されていれば 
warning を出力する Closure クラス)


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
    # ifdef ENABLE_ZAP_DEAD_LOCALS
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
      class CheckValueClosure: public OopClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
frame クラスの _check_value フィールド (static フィールド) に(のみ)格納されている.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* frame::zap_dead_interpreted_locals()
* frame::zap_dead_compiled_locals()

### 備考(Notes)
現状では, ENABLE_ZAP_DEAD_LOCALS は #ifdef ASSERT 時かつ #ifdef COMPILER2 時にしか定義されない.


```cpp
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    // Enable zap-a-lot if in debug version.
    
    # ifdef ASSERT
    # ifdef COMPILER2
    #   define ENABLE_ZAP_DEAD_LOCALS
    #endif /* COMPILER2 */
    # endif /* ASSERT */
```




### 詳細(Details)
See: [here](../doxygen/classframe_1_1CheckValueClosure.html) for details

---
## <a name="noJe44pmYH" id="noJe44pmYH">frame::CheckOopClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef ENABLE_ZAP_DEAD_LOCALS 時にしか定義されない).

スタックフレーム中の oop の格納場所について, 
誤って oop ではない値が格納されていないかどうかをチェックする Closure クラス


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
    # ifdef ENABLE_ZAP_DEAD_LOCALS
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
      class CheckOopClosure: public OopClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
frame クラスの _check_oop フィールド (static フィールド) に(のみ)格納されている.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* frame::check_derived_oop()
* frame::zap_dead_interpreted_locals()
* frame::zap_dead_compiled_locals()

### 備考(Notes)
現状では, ENABLE_ZAP_DEAD_LOCALS は #ifdef ASSERT 時かつ #ifdef COMPILER2 時にしか定義されない.


```cpp
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    // Enable zap-a-lot if in debug version.
    
    # ifdef ASSERT
    # ifdef COMPILER2
    #   define ENABLE_ZAP_DEAD_LOCALS
    #endif /* COMPILER2 */
    # endif /* ASSERT */
```




### 詳細(Details)
See: [here](../doxygen/classframe_1_1CheckOopClosure.html) for details

---
## <a name="noP3Jbsts2" id="noP3Jbsts2">frame::ZapDeadClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef ENABLE_ZAP_DEAD_LOCALS 時にしか定義されない).

スタックフレーム中の dead になっている箇所(= 今後使用されない箇所)について, 
zap 処理 (= 明示的に壊れた値を書き込むことで間違って使用した場合の検出を容易にする処理) を行う.


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
    # ifdef ENABLE_ZAP_DEAD_LOCALS
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
      class ZapDeadClosure: public OopClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
frame クラスの _zap_dead フィールド (static フィールド) に(のみ)格納されている.

#### 使用箇所(where its instances are used)
frame::zap_dead_interpreted_locals() 内で(のみ)使用されている.

### 内部構造(Internal structure)
現在は, 0xbabebabe という値を書き込んでいる.

See: [here](no1904wHW.html) for details
### 備考(Notes)
現状では, ENABLE_ZAP_DEAD_LOCALS は #ifdef ASSERT 時かつ #ifdef COMPILER2 時にしか定義されない.


```cpp
    ((cite: hotspot/src/share/vm/utilities/globalDefinitions.hpp))
    // Enable zap-a-lot if in debug version.
    
    # ifdef ASSERT
    # ifdef COMPILER2
    #   define ENABLE_ZAP_DEAD_LOCALS
    #endif /* COMPILER2 */
    # endif /* ASSERT */
```




### 詳細(Details)
See: [here](../doxygen/classframe_1_1ZapDeadClosure.html) for details

---
## <a name="noshhOwSiZ" id="noshhOwSiZ">FrameValues</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

スタックフレーム内のレイアウトについて, 
レイアウトが正しいかどうかの確認, またはレイアウト情報の出力を行う.


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
    #ifdef ASSERT
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
    // A collection of described stack values that can print a symbolic
    // description of the stack memory.  Interpreter frame values can be
    // in the caller frames so all the values are collected first and then
    // sorted before being printed.
    class FrameValues {
```

### 使われ方(Usage)
JavaThread::print_frame_layout() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classFrameValues.html) for details

---
## <a name="nove92Uhha" id="nove92Uhha">FrameValue</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

FrameValues クラス内で使用される補助クラス.

スタックフレーム内の各値のレイアウト情報を表す.
1つの FrameValue オブジェクトが 1つの値に対応する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
    #ifdef ASSERT
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
    // A simple class to describe a location on the stack
    class FrameValue VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 FrameValues オブジェクトの _values フィールドに(のみ)格納されている.

(正確には, このフィールドは FrameValue の GrowableArray を格納するフィールド.
この中に, その FrameValues 用の全ての FrameValue オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
GrowableArray 用のメモリ領域は FrameValues オブジェクトの生成時に一緒に生成される.

そのメモリ領域中に個別の FrameValue オブジェクトを書き込む作業は FrameValues::describe() 内で(のみ)行われている.

### 内部構造(Internal structure)
単なる構造体のようなクラス.
内部には以下の 4つの public フィールドのみを持つ (そしてメソッドはない).


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
      intptr_t* location;
      char* description;
      int owner;
      int priority;
```




### 詳細(Details)
See: [here](../doxygen/classFrameValue.html) for details

---
## <a name="noUfMWLP1E" id="noUfMWLP1E">StackFrameStream</a>

### 概要(Summary)
指定したスレッドのスタックフレームをたどるためのイテレータクラス(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.hpp))
    // StackFrameStream iterates through the frames of a thread starting from
    // top most frame. It automatically takes care of updating the location of
    // all (callee-saved) registers. Notice: If a thread is stopped at
    // a safepoint, all registers are saved, not only the callee-saved ones.
    //
    // Use:
    //
    //   for(StackFrameStream fst(thread); !fst.is_done(); fst.next()) {
    //     ...
    //   }
    //
    class StackFrameStream : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JavaThread の処理

  * JavaThread::frames_do()
  * JavaThread::oops_do()
  * JavaThread::nmethods_do()

* 脱最適化処理 (Deoptimization 処理)

  * Deoptimization::revoke_biases_of_monitors()
  * Deoptimization::revoke_biases_of_monitors()
  * JavaThread::deoptimized_wrt_marked_nmethods()

* VMError による処理 (print_stack_trace)

  * VMError::print_stack_trace()

* デバッグ用(開発時用)の処理

  * JavaThread::deoptimize()
  * JavaThread::make_zombies()
  * JavaThread::trace_frames()
  * JavaThread::print_frame_layout()
  * VM_DeoptimizeAll::doit()
  * InterfaceSupport::zap_dead_locals_old()
  * InterfaceSupport::stress_derived_pointers()
  * InterfaceSupport::verify_stack()
  * OptoRuntime::zap_dead_java_or_native_locals()




### 詳細(Details)
See: [here](../doxygen/classStackFrameStream.html) for details

---
## <a name="nopQhUUk-u" id="nopQhUUk-u">InterpreterFrameClosure</a>

### 概要(Summary)
OffsetClosure クラスの具象サブクラスの1つ.

他の OopClosure と組み合わせて使用される Closure クラス.
Interpreter フレーム内(局所変数領域内／オペランドスタック内)に現在存在する全ての oop に対して, 
指定された OopClosure を適用する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.cpp))
    /*
      The interpreter_frame_expression_stack_at method in the case of SPARC needs the
      max_stack value of the method in order to compute the expression stack address.
      It uses the methodOop in order to get the max_stack value but during GC this
      methodOop value saved on the frame is changed by reverse_and_push and hence cannot
      be used. So we save the max_stack value in the FrameClosure object and pass it
      down to the interpreter_frame_expression_stack_at method
    */
    class InterpreterFrameClosure : public OffsetClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* frame::oops_interpreted_do()
* frame::zap_dead_interpreted_locals()




### 詳細(Details)
See: [here](../doxygen/classInterpreterFrameClosure.html) for details

---
## <a name="nonGOYNKud" id="nonGOYNKud">InterpretedArgumentOopFinder</a>

### 概要(Summary)
スタックフレームに対する oops_do 処理 (= frame::oops_do()) 用の補助クラス(SignatureInfoクラス).
なお, このクラスは他の OopClosure と組み合わせて使用される.

Safepoint 処理で停止した地点がメソッドの呼び出し点(invoke* バイトコード等)であれば, 
引数中に含まれる全ての oop についても処理を行う必要がある.
このクラスは, 引数中の全ての oop に対して, 指定された OopClosure を適用する.

(なお, フレームの種別に応じて, 同様の役割を持つクラスが幾つか存在する.
 このクラスは Interpreter のフレーム用 (See: EntryFrameOopFinder, CompiledArgumentOopFinder))


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.cpp))
    class InterpretedArgumentOopFinder: public SignatureInfo {
```

### 使われ方(Usage)
frame::oops_interpreted_arguments_do() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classInterpretedArgumentOopFinder.html) for details

---
## <a name="noC0SULiUS" id="noC0SULiUS">EntryFrameOopFinder</a>

### 概要(Summary)
スタックフレームに対する oops_do 処理 (= frame::oops_do()) 用の補助クラス(SignatureInfoクラス).
なお, このクラスは他の OopClosure と組み合わせて使用される.

Safepoint 処理で停止した地点がメソッドの呼び出し点(invoke* バイトコード等)であれば, 
引数中に含まれる全ての oop についても処理を行う必要がある.
このクラスは, 引数中の全ての oop に対して, 指定された OopClosure を適用する.

(なお, フレームの種別に応じて, 同様の役割を持つクラスが幾つか存在する.
 このクラスは entry frame (JavaCalls 経由で Java コードを呼んだときに積まれるダミーのフレーム) 用 
 (See: InterpretedArgumentOopFinder, CompiledArgumentOopFinder))


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.cpp))
    // Entry frame has following form (n arguments)
    //         +-----------+
    //   sp -> |  last arg |
    //         +-----------+
    //         :    :::    :
    //         +-----------+
    // (sp+n)->|  first arg|
    //         +-----------+
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.cpp))
    // visits and GC's all the arguments in entry frame
    class EntryFrameOopFinder: public SignatureInfo {
```

### 使われ方(Usage)
frame::oops_entry_do() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classEntryFrameOopFinder.html) for details

---
## <a name="noxNORMtNS" id="noxNORMtNS">CompiledArgumentOopFinder</a>

### 概要(Summary)
スタックフレームに対する oops_do 処理 (= frame::oops_do()) 用の補助クラス(SignatureInfoクラス).
なお, このクラスは他の OopClosure と組み合わせて使用される.

Safepoint 処理で停止した地点がメソッドの呼び出し点(invoke* バイトコード等)であれば, 
引数中に含まれる全ての oop についても処理を行う必要がある.
このクラスは, 引数中の全ての oop に対して, 指定された OopClosure を適用する.

(なお, フレームの種別に応じて, 同様の役割を持つクラスが幾つか存在する.
 このクラスは JIT 生成コードのフレーム用 (See: InterpretedArgumentOopFinder, EntryFrameOopFinder))


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.cpp))
    class CompiledArgumentOopFinder: public SignatureInfo {
```

### 使われ方(Usage)
frame::oops_compiled_arguments_do() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classCompiledArgumentOopFinder.html) for details

---
