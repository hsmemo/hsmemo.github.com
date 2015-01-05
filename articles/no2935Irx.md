---
layout: default
title: StubGenerator クラス 
---
[Top](../index.html)

#### StubGenerator クラス 



---
## <a name="no_w4kXsvD" id="no_w4kXsvD">StubGenerator</a>

### 概要(Summary)
StubCodeGenerator クラスの具象サブクラスの1つ.
このクラスは StubRoutines 用のコードを生成する.

(なお, 32bit か 64bit かによってクラス定義が別になっている.)


```cpp
    ((cite: hotspot/src/cpu/x86/vm/stubGenerator_x86_32.cpp))
    class StubGenerator: public StubCodeGenerator {
```


```cpp
    ((cite: hotspot/src/cpu/x86/vm/stubGenerator_x86_64.cpp))
    class StubGenerator: public StubCodeGenerator {
```

### 使われ方(Usage)
StubGenerator_generate() 内で(のみ)使用されている.

なお, StubRoutine 内のコード生成は以下のように2段階で行われる
(universe の初期化との関係上, 一度に全部やると卵と鶏の問題が起こってしまうため.
 順番としては, まず stubRoutines_init1() を行った後で universe::genesis(universe の初期化)を行い, 
 改めて stubRoutines_init2() を行う).

<div class="flow-abst"><pre>
JNI_CreateJavaVM()
-&gt; (HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
   -&gt; init_globals()
      ...
      -&gt; stubRoutines_init1()
         -&gt; StubRoutines::initialize1()
            -&gt; StubGenerator_generate()          (第2引数 false で呼び出される)
               -&gt; StubGenerator::StubGenerator() (第2引数 false で呼び出される)
                  -&gt; StubGenerator::generate_initial()
      -&gt; universe_init()
      ...
      -&gt; stubRoutines_init2()
         -&gt; StubRoutines::initialize2()
            -&gt; StubGenerator_generate()          (第2引数 true で呼び出される)
               -&gt; StubGenerator::StubGenerator() (第2引数 true で呼び出される)
                  -&gt; StubGenerator::generate_all()
</pre></div>

#### 参考(for your information): StubGenerator_generate() (x86 32bit の場合)
See: [here](no2935H_G.html) for details
#### 参考(for your information): StubGenerator::StubGenerator() (x86 32bit の場合)
See: [here](no2935UJN.html) for details
#### 参考(for your information): StubGenerator::generate_initial() (x86 32bit の場合)
(#TODO)
See: [here](no29357nf.html) for details
#### 参考(for your information): StubGenerator::generate_all() (x86 32bit の場合)
(#TODO)
See: [here](no2935Iyl.html) for details

#### 参考(for your information): StubGenerator_generate() (x86 64bit の場合)
See: [here](no2935udZ.html) for details
#### 参考(for your information): StubGenerator::StubGenerator() (x86 64bit の場合)
See: [here](no2935hTT.html) for details
#### 参考(for your information): StubGenerator::generate_initial() (x86 64bit の場合)
(#TODO)
See: [here](no2935iGy.html) for details
#### 参考(for your information): StubGenerator::generate_all() (x86 64bit の場合)
(#TODO)
See: [here](no2935V8r.html) for details

### 内部構造(Internal structure)
内部には, 以下のようなメソッドが定義されている.

  * StubGenerator::generate_call_stub()

    -- VMルーチンから Java メソッドを呼び出す際のスタブ作成用のコードを生成 (See: [here](no3059iJu.html) for details)

  * StubGenerator::generate_catch_exception()

    -- VM 内部での例外catch処理を行うコードを生成 (See: [here](no3059dEL.html) for details)

  * StubGenerator::generate_forward_exception()

    -- 例外ハンドラに分岐するコード(exception handler を取得してそこにジャンプするコード)を生成 (See: [here](no293560A.html) and [here](no3059d4M.html) for details)

  * StubGenerator::generate_throw_exception()

    -- 例外ハンドリングを開始させるコードを生成. より具体的には, 引数で与えられた entrypoint を呼び出した後, StubRoutines::forward_exception_entry() (= generate_forward_exception() が作ったコード) へジャンプする (See: [here](no29358hy.html) for details).

    (なお, Template Interpreter の例外処理では TemplateInterpreterGenerator::generate_throw_exception() が生成したコードが使われる. こちらは Template Interpreter 以外の場合用)

  * StubGenerator::generate_atomic_xchg()

    -- アトミックなメモリ操作用のコードを生成

  * StubGenerator::generate_atomic_xchg_ptr()

    -- (64bit の場合にのみ存在) 同上

  * StubGenerator::generate_atomic_cmpxchg()

    -- (64bit の場合にのみ存在) 同上

  * StubGenerator::generate_atomic_cmpxchg_long()

    -- (64bit の場合にのみ存在) 同上

  * StubGenerator::generate_atomic_add()

    -- (64bit の場合にのみ存在) 同上

  * StubGenerator::generate_atomic_add_ptr()

    -- (64bit の場合にのみ存在) 同上

  * StubGenerator::generate_orderaccess_fence()

    -- (64bit の場合にのみ存在) メモリバリア命令を実行するコードを生成

  * StubGenerator::generate_get_previous_fp()
  * StubGenerator::generate_verify_mxcsr()
  * StubGenerator::generate_f2i_fixup()
  * StubGenerator::generate_f2l_fixup()
  * StubGenerator::generate_d2i_fixup()
  * StubGenerator::generate_d2l_fixup()
  * StubGenerator::generate_fp_mask()
  * StubGenerator::generate_handler_for_unsafe_access()
    
    -- sun.misc.Unsafe のメソッド内で起きたメモリアクセス違反をハンドリングするためのコードを生成 (See: [here](no7882SOc.html) for details)

  * StubGenerator::generate_verify_oop()
  * StubGenerator::array_overlap_test()
  * StubGenerator::array_overlap_test()
  * StubGenerator::array_overlap_test()
  * StubGenerator::setup_arg_regs()
  * StubGenerator::restore_arg_regs()
  * StubGenerator::gen_write_ref_array_pre_barrier()
  * StubGenerator::gen_write_ref_array_post_barrier()
  * StubGenerator::copy_32_bytes_forward()
  * StubGenerator::copy_32_bytes_backward()
  * StubGenerator::generate_disjoint_byte_copy()
  * StubGenerator::generate_conjoint_byte_copy()
  * StubGenerator::generate_disjoint_short_copy()
  * StubGenerator::generate_fill()
  * StubGenerator::generate_conjoint_short_copy()
  * StubGenerator::generate_disjoint_int_oop_copy()
  * StubGenerator::generate_conjoint_int_oop_copy()
  * StubGenerator::generate_disjoint_long_oop_copy()
  * StubGenerator::generate_conjoint_long_oop_copy()
  * StubGenerator::generate_type_check()
  * StubGenerator::generate_checkcast_copy()
  * StubGenerator::generate_unsafe_copy()
  * StubGenerator::arraycopy_range_checks()
  * StubGenerator::generate_generic_copy()
  * StubGenerator::generate_arraycopy_stubs()
  * StubGenerator::generate_math_stubs()




### 詳細(Details)
See: [here](../doxygen/classStubGenerator.html) for details

---
