---
layout: default
title: Deoptimization クラス関連のクラス (Deoptimization, Deoptimization::UnrollBlock, DeoptimizationMarker, 及びそれらの補助クラス(FieldReassigner))
---
[Top](../index.html)

#### Deoptimization クラス関連のクラス (Deoptimization, Deoptimization::UnrollBlock, DeoptimizationMarker, 及びそれらの補助クラス(FieldReassigner))

これらは, JIT Compiler 用のクラス.
より具体的に言うと, 脱最適化処理で使用されるクラス
(See: [here](no3420xYb.html) for details).


### クラス一覧(class list)

  * [Deoptimization](#no-ZFJW6fh)
  * [Deoptimization::UnrollBlock](#nogvhPqSSv)
  * [DeoptimizationMarker](#no7R60-mE_)
  * [FieldReassigner](#nogvFwuPdb)


---
## <a name="no-ZFJW6fh" id="no-ZFJW6fh">Deoptimization</a>

### 概要(Summary)
脱最適化処理(Deoptimization)を行うためのクラス 
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)) 
(See: [here](no3420xYb.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.hpp))
    class Deoptimization : AllStatic {
```

### 内部構造(Internal structure)
内部には定数定義およびメソッド定義(のみ)を含む.

定義されている定数値は, 以下の通り.

  * Deoptimization::DeoptReason
    
    脱最適化処理を引き起こした原因を表す定数型.

  * Deoptimization::DeoptAction
    
    脱最適化処理で行うべき処理を表す定数型.

  * (enum 型名なし)

    DeoptReason 型, および DeoptAction 型のビット数を表す定数値.

  * Deoptimization::UnpackType

    脱最適化処理後に処理を引き継ぐ先を示す定数型
    (See: vframeArrayElement::unpack_on_stack())


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.hpp))
      // What condition caused the deoptimization?
      enum DeoptReason {
        Reason_many = -1,             // indicates presence of several reasons
        Reason_none = 0,              // indicates absence of a relevant deopt.
        // Next 7 reasons are recorded per bytecode in DataLayout::trap_bits
        Reason_null_check,            // saw unexpected null or zero divisor (@bci)
        Reason_null_assert,           // saw unexpected non-null or non-zero (@bci)
        Reason_range_check,           // saw unexpected array index (@bci)
        Reason_class_check,           // saw unexpected object class (@bci)
        Reason_array_check,           // saw unexpected array class (aastore @bci)
        Reason_intrinsic,             // saw unexpected operand to intrinsic (@bci)
        Reason_bimorphic,             // saw unexpected object class in bimorphic inlining (@bci)
    
        Reason_unloaded,              // unloaded class or constant pool entry
        Reason_uninitialized,         // bad class state (uninitialized)
        Reason_unreached,             // code is not reached, compiler
        Reason_unhandled,             // arbitrary compiler limitation
        Reason_constraint,            // arbitrary runtime constraint violated
        Reason_div0_check,            // a null_check due to division by zero
        Reason_age,                   // nmethod too old; tier threshold reached
        Reason_predicate,             // compiler generated predicate failed
        Reason_loop_limit_check,      // compiler generated loop limits check failed
        Reason_LIMIT,
        // Note:  Keep this enum in sync. with _trap_reason_name.
        Reason_RECORDED_LIMIT = Reason_bimorphic  // some are not recorded per bc
        // Note:  Reason_RECORDED_LIMIT should be < 8 to fit into 3 bits of
        // DataLayout::trap_bits.  This dependency is enforced indirectly
        // via asserts, to avoid excessive direct header-to-header dependencies.
        // See Deoptimization::trap_state_reason and class DataLayout.
      };
    
      // What action must be taken by the runtime?
      enum DeoptAction {
        Action_none,                  // just interpret, do not invalidate nmethod
        Action_maybe_recompile,       // recompile the nmethod; need not invalidate
        Action_reinterpret,           // invalidate the nmethod, reset IC, maybe recompile
        Action_make_not_entrant,      // invalidate the nmethod, recompile (probably)
        Action_make_not_compilable,   // invalidate the nmethod and do not compile
        Action_LIMIT
        // Note:  Keep this enum in sync. with _trap_action_name.
      };
    
      enum {
        _action_bits = 3,
        _reason_bits = 5,
        _action_shift = 0,
        _reason_shift = _action_shift+_action_bits,
        BC_CASE_LIMIT = PRODUCT_ONLY(1) NOT_PRODUCT(4) // for _deoptimization_hist
      };
    
      enum UnpackType {
        Unpack_deopt                = 0, // normal deoptimization, use pc computed in unpack_vframe_on_stack
        Unpack_exception            = 1, // exception is pending
        Unpack_uncommon_trap        = 2, // redo last byte code (C2 only)
        Unpack_reexecute            = 3  // reexecute bytecode (C1 only)
      };
```

定義されている public メソッドは, 以下の通り.


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.hpp))
      // Checks all compiled methods. Invalid methods are deleted and
      // corresponding activations are deoptimized.
      static int deoptimize_dependents();
    
      // Deoptimizes a frame lazily. nmethod gets patched deopt happens on return to the frame
      static void deoptimize(JavaThread* thread, frame fr, RegisterMap *reg_map);
```


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.hpp))
      static vframeArray* create_vframeArray(JavaThread* thread, frame fr, RegisterMap *reg_map, GrowableArray<compiledVFrame*>* chunk);
```


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.hpp))
      //** Returns an UnrollBlock continuing information
      // how to make room for the resulting interpreter frames.
      // Called by assembly stub after execution has returned to
      // deoptimized frame.
      // @argument thread.     Thread where stub_frame resides.
      // @see OptoRuntime::deoptimization_fetch_unroll_info_C
      static UnrollBlock* fetch_unroll_info(JavaThread* thread);
    
      //** Unpacks vframeArray onto execution stack
      // Called by assembly stub after execution has returned to
      // deoptimized frame and after the stack unrolling.
      // @argument thread.     Thread where stub_frame resides.
      // @argument exec_mode.  Determines how execution should be continuted in top frame.
      //                       0 means continue after current byte code
      //                       1 means exception has happened, handle exception
      //                       2 means reexecute current bytecode (for uncommon traps).
      // @see OptoRuntime::deoptimization_unpack_frames_C
      // Return BasicType of call return type, if any
      static BasicType unpack_frames(JavaThread* thread, int exec_mode);
    
      // Cleans up deoptimization bits on thread after unpacking or in the
      // case of an exception.
      static void cleanup_deopt_info(JavaThread  *thread,
                                     vframeArray * array);
    
      // Restores callee saved values from deoptimized frame into oldest interpreter frame
      // so caller of the deoptimized frame will get back the values it expects.
      static void unwind_callee_save_values(frame* f, vframeArray* vframe_array);
    
      //** Performs an uncommon trap for compiled code.
      // The top most compiler frame is converted into interpreter frames
      static UnrollBlock* uncommon_trap(JavaThread* thread, jint unloaded_class_index);
      // Helper routine that enters the VM and may block
      static void uncommon_trap_inner(JavaThread* thread, jint unloaded_class_index);
    
      //** Deoptimizes the frame identified by id.
      // Only called from VMDeoptimizeFrame
      // @argument thread.     Thread where stub_frame resides.
      // @argument id.         id of frame that should be deoptimized.
      static void deoptimize_frame_internal(JavaThread* thread, intptr_t* id);
    
      // If thread is not the current thread then execute
      // VM_DeoptimizeFrame otherwise deoptimize directly.
      static void deoptimize_frame(JavaThread* thread, intptr_t* id);
    
      // Statistics
      static void gather_statistics(DeoptReason reason, DeoptAction action,
                                    Bytecodes::Code bc = Bytecodes::_illegal);
      static void print_statistics();
    
      // How much room to adjust the last frame's SP by, to make space for
      // the callee's interpreter frame (which expects locals to be next to
      // incoming arguments)
      static int last_frame_adjust(int callee_parameters, int callee_locals);
    
      // trap_request codes
      static DeoptReason trap_request_reason(int trap_request) {
    ...
      }
      static DeoptAction trap_request_action(int trap_request) {
    ...
      }
      static int trap_request_index(int trap_request) {
    ...
      }
      static int make_trap_request(DeoptReason reason, DeoptAction action,
                                   int index = -1) {
    ...
      }
    
      // The trap_state stored in a MDO is decoded here.
      // It records two items of information.
      //  reason:  If a deoptimization happened here, what its reason was,
      //           or if there were multiple deopts with differing reasons.
      //  recompiled: If a deoptimization here triggered a recompilation.
      // Note that not all reasons are recorded per-bci.
      static DeoptReason trap_state_reason(int trap_state);
      static int  trap_state_has_reason(int trap_state, int reason);
      static int  trap_state_add_reason(int trap_state, int reason);
      static bool trap_state_is_recompiled(int trap_state);
      static int  trap_state_set_recompiled(int trap_state, bool z);
      static const char* format_trap_state(char* buf, size_t buflen,
                                           int trap_state);
    
      static bool reason_is_recorded_per_bytecode(DeoptReason reason) {
    ...
      }
    
      static DeoptReason reason_recorded_per_bytecode_if_any(DeoptReason reason) {
    ...
      }
    
      static const char* trap_reason_name(int reason);
      static const char* trap_action_name(int action);
      // Format like reason='foo' action='bar' index='123'.
      // This is suitable both for XML and for tty output.
      static const char* format_trap_request(char* buf, size_t buflen,
                                             int trap_request);
    
      static jint total_deoptimization_count();
      static jint deoptimization_count(DeoptReason reason);
    
      // JVMTI PopFrame support
    
      // Preserves incoming arguments to the popped frame when it is
      // returning to a deoptimized caller
      static void popframe_preserve_args(JavaThread* thread, int bytes_to_save, void* start_address);
```


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.hpp))
      static void update_method_data_from_interpreter(methodDataHandle trap_mdo, int trap_bci, int reason);
```




### 詳細(Details)
See: [here](../doxygen/classDeoptimization.html) for details

---
## <a name="nogvhPqSSv" id="nogvhPqSSv">Deoptimization::UnrollBlock</a>

### 概要(Summary)
Deoptimization クラス内で使用される補助クラス.

脱最適化対象となったスタックフレームの情報を記録しておくためのクラス.
これらの情報は, 脱最適化で Interpreter フレームを構築する処理, 
および構築後に Interpreter フレーム内の値を埋める際に使用される.

(なお, 構築後に Interpreter フレーム内の値を埋める処理は vframeArray クラスと組み合わせて使用される)

(なお, vframeArray クラスと同様, CHeapObj クラスになっているが本質的には ResourceObj クラスにしても問題ない.
 後から中身を参照できるとデバッグ時に便利なので CHeapObj となっているだけ).


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.hpp))
      // Interface used for unpacking deoptimized frames
    
      // UnrollBlock is returned by fetch_unroll_info() to the deoptimization handler (blob).
      // This is only a CheapObj to ease debugging after a deopt failure
      class UnrollBlock : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 vframeArray オブジェクトの _unroll_block フィールドに(のみ)格納されている.
 
ただし, Deoptimization::fetch_unroll_info() および 
Deoptimization::uncommon_trap() の返値としても返されており, 
この値が呼び出し元のフレーム内で局所的に使用されている.
具体的に言うと, 以下のレジスタ内に格納されている.

* SharedRuntime::generate_deopt_blob() が生成するコード
  
  * O2 レジスタ (ソース中では O2UnrollBlock レジスタ) (Sparc の場合)
  * RDI レジスタ (x86_64 の場合)

* SharedRuntime::generate_uncommon_trap_blob() が生成するコード
  
  * O2 レジスタ (ソース中では O2UnrollBlock レジスタ) (Sparc の場合)
  * RDI レジスタ (x86_64 の場合)

#### 生成箇所(where its instances are created)
Deoptimization::fetch_unroll_info_helper() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* Interpreter フレームを構築する処理 (上記のレジスタ経由で使用される)

    * Sparc の場合
      
        * SharedRuntime::generate_deopt_blob() が生成するコード
          
          (正確にはその中で呼ばれる make_new_frames() の中)
      
        * SharedRuntime::generate_uncommon_trap_blob() が生成するコード
          
          (正確にはその中で呼ばれる make_new_frames() の中)
      
    * x86_64 の場合

        * SharedRuntime::generate_deopt_blob() が生成するコード
      
        * SharedRuntime::generate_uncommon_trap_blob() が生成するコード

* Interpreter フレーム内の値を埋める処理 (vframeArray::_unroll_block フィールド経由で使用される)
  
    * Deoptimization::unpack_frames()


#### 削除箇所(where its instances are deleted)
以下の箇所で(のみ)削除されている.

* Deoptimization::cleanup_deopt_info()
  
  (より正確に言うと, JavaThread::_vframe_array_last フィールドの vframeArray が削除され, 
  JavaThread::_vframe_array_head フィールドの vframeArray が JavaThread::_vframe_array_last フィールドに移される)

* JavaThread::~JavaThread()
  
  (より正確に言うと, JavaThread::_vframe_array_last フィールドの vframeArray が削除される)




### 詳細(Details)
See: [here](../doxygen/classDeoptimization_1_1UnrollBlock.html) for details

---
## <a name="no7R60-mE_" id="no7R60-mE_">DeoptimizationMarker</a>

### 概要(Summary)
保守運用機能のためのクラス (関連するオプションが指定されている場合にのみ使用される) (See: -Xprof).

FlatProfiler クラス用の補助クラス. 現在 Deoptimization 処理(脱最適化処理)が実行中かどうかを示す.

(より具体的に言うと, ソースコード中のあるスコープの間だけ, 
 DeoptimizationMarker::_is_active という static フィールドの値を true にするためのクラス(StackObjクラス))

(FlatProfiler は定期的に統計情報のサンプリング処理を行う.
その処理の1つに, サンプリング時に脱最適化処理が実行中であれば 
FlatProfiler::deopt_ticks フィールドをインクリメントするというものがある.
DeoptimizationMarker はこれを実現するためのクラス 
(See: FlatProfiler::record_vm_operation()))


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.hpp))
    class DeoptimizationMarker : StackObj {  // for profiling
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で DeoptimizationMarker 型の局所変数を宣言するだけ.

(基本的には脱最適化処理を行う直前で宣言すればいい)

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* Universe::flush_dependents_on_method()
* Universe::flush_evol_dependents_on()
* VM_RedefineClasses::flush_dependent_code()
* Deoptimization::deoptimize()
* VM_Deoptimize::doit()
* VM_DeoptimizeAll::doit()
* VM_DeoptimizeTheWorld::doit()

### 内部構造(Internal structure)
コンストラクタで DeoptimizationMarker::_is_active を true にセットし, 
デストラクタで false に戻すだけ.


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.hpp))
      DeoptimizationMarker()  { _is_active = true; }
      ~DeoptimizationMarker() { _is_active = false; }
```




### 詳細(Details)
See: [here](../doxygen/classDeoptimizationMarker.html) for details

---
## <a name="nogvFwuPdb" id="nogvFwuPdb">FieldReassigner</a>

### 概要(Summary)
Deoptimization クラス内で使用される補助クラス(Closureクラス).

脱最適化処理の際に, 
エスケープ解析によってスタックフレーム上に確保されたオブジェクトについて, 
そのフィールドの値をヒープ上に確保したオブジェクト内に移し替えるための Closure クラス.


```
    ((cite: hotspot/src/share/vm/runtime/deoptimization.cpp))
    // This assumes that the fields are stored in ObjectValue in the same order
    // they are yielded by do_nonstatic_fields.
    class FieldReassigner: public FieldClosure {
```

### 使われ方(Usage)
Deoptimization::reassign_fields() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classFieldReassigner.html) for details

---
