---
layout: default
title: StubRoutines クラス 
---
[Top](../index.html)

#### StubRoutines クラス 



---
## <a name="noOZBn9GO3" id="noOZBn9GO3">StubRoutines</a>

### 概要(Summary)
HotSpot 内で (インタープリタや JIT 生成コード, ランタイムなどから) 
共通で使われるマシン語コード片を納めておくための名前空間(AllStatic クラス).

例えば, 以下のような処理ルーチン用のコードが格納されている.

* Java のメソッドを呼び出す処理ルーチン (StubRoutines::call_stub())
* 例外ハンドリングを行う処理ルーチン (StubRoutines::forward_exception_entry(), StubRoutines::throw_*_entry())
* アトミックなメモリ書き換えを行う処理ルーチン (StubRoutines::atomic_*())
* メモリコピーを行う処理ルーチン (StubRoutines::*_arraycopy())

なお, 実際にこれらのマシン語コードを生成するのは StubGenerator クラス.
StubRoutines クラスは, 生成されたマシン語コード片へのポインタを格納しているだけのクラス.

(この StubGenerator クラスは cpu/ 下の stubGenerator_${arch}.cpp で定義されている.
 現状ではプラットフォーム非依存な内容までそこに含まれているが, 
 1ファイルで完結している方が嬉しいこともあるのでとりあえずそのままになっている, とのこと)

なお StubRoutines は複数のファイルにまたがって定義されている. それらの関係は以下の通り.

* プラットフォーム非依存な部分は, stubRoutines.hpp で宣言され, stubRoutines(_*).cpp で定義.

* プラットフォーム依存な部分は, stubRoutines_${arch}.hpp で宣言され, 
  stubRoutines_${arch}.cpp や stubGenerator_${arch}.cpp で定義.


        platform-independent               platform-dependent

        stubRoutines.hpp  <-- included --  stubRoutines_<arch>.hpp
               ^                                  ^
               |                                  |
           implements                         implements
               |                                  |
               |                                  |
        stubRoutines.cpp                   stubRoutines_<arch>.cpp
        stubRoutines_<os_family>.cpp       stubGenerator_<arch>.cpp
        stubRoutines_<os_arch>.cpp


新しいマシン語コードを追加する際には, 以下のようにするといい.

1. プラットフォーム非依存かどうかに応じて変更するファイルを適切に選択.
2. address 型のフィールドを追加.
3. そのフィールドへのアクセサメソッドを定義.
4. stubGenerator_${arch}.cpp にアセンブリルーチンを作成するメソッドを追加し, 
   generate_all() から追加したメソッドを呼ぶ.


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubRoutines.hpp))
    // StubRoutines provides entry points to assembly routines used by
    // compiled code and the run-time system. Platform-specific entry
    // points are defined in the platform-specific inner class.
    //
    // Class scheme:
    //
    //    platform-independent               platform-dependent
    //
    //    stubRoutines.hpp  <-- included --  stubRoutines_<arch>.hpp
    //           ^                                  ^
    //           |                                  |
    //       implements                         implements
    //           |                                  |
    //           |                                  |
    //    stubRoutines.cpp                   stubRoutines_<arch>.cpp
    //    stubRoutines_<os_family>.cpp       stubGenerator_<arch>.cpp
    //    stubRoutines_<os_arch>.cpp
    //
    // Note 1: The important thing is a clean decoupling between stub
    //         entry points (interfacing to the whole vm; i.e., 1-to-n
    //         relationship) and stub generators (interfacing only to
    //         the entry points implementation; i.e., 1-to-1 relationship).
    //         This significantly simplifies changes in the generator
    //         structure since the rest of the vm is not affected.
    //
    // Note 2: stubGenerator_<arch>.cpp contains a minimal portion of
    //         machine-independent code; namely the generator calls of
    //         the generator functions that are used platform-independently.
    //         However, it comes with the advantage of having a 1-file
    //         implementation of the generator. It should be fairly easy
    //         to change, should it become a problem later.
    //
    // Scheme for adding a new entry point:
    //
    // 1. determine if it's a platform-dependent or independent entry point
    //    a) if platform independent: make subsequent changes in the independent files
    //    b) if platform   dependent: make subsequent changes in the   dependent files
    // 2. add a private instance variable holding the entry point address
    // 3. add a public accessor function to the instance variable
    // 4. implement the corresponding generator function in the platform-dependent
    //    stubGenerator_<arch>.cpp file and call the function in generate_all() of that file
    
    
    class StubRoutines: AllStatic {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
#### 定義されているフィールド
定義されているフィールドは以下の通り (address 型のフィールドに StubGenerator が生成したコードへのポインタが格納される).

(share 部で定義されているフィールドは, 以下のように address 型のものがほとんど)


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubRoutines.hpp))
      static jint    _verify_oop_count;
      static address _verify_oop_subroutine_entry;
    
      static address _call_stub_return_address;                // the return PC, when returning to a call stub
      static address _call_stub_entry;
      static address _forward_exception_entry;
      static address _catch_exception_entry;
      static address _throw_AbstractMethodError_entry;
      static address _throw_IncompatibleClassChangeError_entry;
      static address _throw_ArithmeticException_entry;
      static address _throw_NullPointerException_entry;
      static address _throw_NullPointerException_at_call_entry;
      static address _throw_StackOverflowError_entry;
      static address _throw_WrongMethodTypeException_entry;
      static address _handler_for_unsafe_access_entry;
    
      static address _atomic_xchg_entry;
      static address _atomic_xchg_ptr_entry;
      static address _atomic_store_entry;
      static address _atomic_store_ptr_entry;
      static address _atomic_cmpxchg_entry;
      static address _atomic_cmpxchg_ptr_entry;
      static address _atomic_cmpxchg_long_entry;
      static address _atomic_add_entry;
      static address _atomic_add_ptr_entry;
      static address _fence_entry;
      static address _d2i_wrapper;
      static address _d2l_wrapper;
    
      static jint    _fpu_cntrl_wrd_std;
      static jint    _fpu_cntrl_wrd_24;
      static jint    _fpu_cntrl_wrd_64;
      static jint    _fpu_cntrl_wrd_trunc;
      static jint    _mxcsr_std;
      static jint    _fpu_subnormal_bias1[3];
      static jint    _fpu_subnormal_bias2[3];
    
      static BufferBlob* _code1;                               // code buffer for initial routines
      static BufferBlob* _code2;                               // code buffer for all other routines
    
      // Leaf routines which implement arraycopy and their addresses
      // arraycopy operands aligned on element type boundary
      static address _jbyte_arraycopy;
      static address _jshort_arraycopy;
      static address _jint_arraycopy;
      static address _jlong_arraycopy;
      static address _oop_arraycopy, _oop_arraycopy_uninit;
      static address _jbyte_disjoint_arraycopy;
      static address _jshort_disjoint_arraycopy;
      static address _jint_disjoint_arraycopy;
      static address _jlong_disjoint_arraycopy;
      static address _oop_disjoint_arraycopy, _oop_disjoint_arraycopy_uninit;
    
      // arraycopy operands aligned on zero'th element boundary
      // These are identical to the ones aligned aligned on an
      // element type boundary, except that they assume that both
      // source and destination are HeapWord aligned.
      static address _arrayof_jbyte_arraycopy;
      static address _arrayof_jshort_arraycopy;
      static address _arrayof_jint_arraycopy;
      static address _arrayof_jlong_arraycopy;
      static address _arrayof_oop_arraycopy, _arrayof_oop_arraycopy_uninit;
      static address _arrayof_jbyte_disjoint_arraycopy;
      static address _arrayof_jshort_disjoint_arraycopy;
      static address _arrayof_jint_disjoint_arraycopy;
      static address _arrayof_jlong_disjoint_arraycopy;
      static address _arrayof_oop_disjoint_arraycopy, _arrayof_oop_disjoint_arraycopy_uninit;
    
      // these are recommended but optional:
      static address _checkcast_arraycopy, _checkcast_arraycopy_uninit;
      static address _unsafe_arraycopy;
      static address _generic_arraycopy;
    
      static address _jbyte_fill;
      static address _jshort_fill;
      static address _jint_fill;
      static address _arrayof_jbyte_fill;
      static address _arrayof_jshort_fill;
      static address _arrayof_jint_fill;
    
      // These are versions of the java.lang.Math methods which perform
      // the same operations as the intrinsic version.  They are used for
      // constant folding in the compiler to ensure equivalence.  If the
      // intrinsic version returns the same result as the strict version
      // then they can be set to the appropriate function from
      // SharedRuntime.
      static double (*_intrinsic_log)(double);
      static double (*_intrinsic_log10)(double);
      static double (*_intrinsic_exp)(double);
      static double (*_intrinsic_pow)(double, double);
      static double (*_intrinsic_sin)(double);
      static double (*_intrinsic_cos)(double);
      static double (*_intrinsic_tan)(double);
```

(以下は Sparc 版専用のフィールド. 同じく address 型のものばかり)


```cpp
    ((cite: hotspot/src/cpu/sparc/vm/stubRoutines_sparc.hpp))
      static address _test_stop_entry;
      static address _stop_subroutine_entry;
      static address _flush_callers_register_windows_entry;
    
      static int _atomic_memory_operation_lock;
    
      static address _partial_subtype_check;
```

(以下は x86-64 版専用のフィールド. こちらも同じく address 型のものばかり)


```cpp
    ((cite: hotspot/src/cpu/x86/vm/stubRoutines_x86_64.hpp))
      static address _get_previous_fp_entry;
      static address _verify_mxcsr_entry;
    
      static address _f2i_fixup;
      static address _f2l_fixup;
      static address _d2i_fixup;
      static address _d2l_fixup;
    
      static address _float_sign_mask;
      static address _float_sign_flip;
      static address _double_sign_mask;
      static address _double_sign_flip;
      static address _mxcsr_std;
```

#### 内部の処理
上で書いた通り, ほとんどの実装は stubGenerator_${arch}.cpp で行われている.

ただし, arraycopy 系のメソッドのデフォルト実装や, 
適切な fill/arraycopy メソッドを選択するためのメソッドについては, 
share 部で定義されていたりする
(といっても arraycopy 系の実際の処理は Copy クラスに丸投げだが...).


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubRoutines.cpp))
    //
    // Default versions of arraycopy functions
    //
    
    static void gen_arraycopy_barrier_pre(oop* dest, size_t count, bool dest_uninitialized) {
        assert(count != 0, "count should be non-zero");
        assert(count <= (size_t)max_intx, "count too large");
        BarrierSet* bs = Universe::heap()->barrier_set();
        assert(bs->has_write_ref_array_pre_opt(), "Must have pre-barrier opt");
        bs->write_ref_array_pre(dest, (int)count, dest_uninitialized);
    }
    
    static void gen_arraycopy_barrier(oop* dest, size_t count) {
        assert(count != 0, "count should be non-zero");
        BarrierSet* bs = Universe::heap()->barrier_set();
        assert(bs->has_write_ref_array_opt(), "Barrier set must have ref array opt");
        bs->write_ref_array((HeapWord*)dest, count);
    }
    
    JRT_LEAF(void, StubRoutines::jbyte_copy(jbyte* src, jbyte* dest, size_t count))
    #ifndef PRODUCT
      SharedRuntime::_jbyte_array_copy_ctr++;      // Slow-path byte array copy
    #endif // !PRODUCT
      Copy::conjoint_jbytes_atomic(src, dest, count);
    JRT_END
    
    ...
    
    address StubRoutines::select_fill_function(BasicType t, bool aligned, const char* &name) {
    ...
    // Note:  The condition "disjoint" applies also for overlapping copies
    // where an descending copy is permitted (i.e., dest_offset <= src_offset).
    address
    StubRoutines::select_arraycopy_function(BasicType t, bool aligned, bool disjoint, const char* &name, bool dest_uninitialized) {
    ...
```




### 詳細(Details)
See: [here](../doxygen/classStubRoutines.html) for details

---
