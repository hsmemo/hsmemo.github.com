---
layout: default
title: StubCodeGenerator クラス関連のクラス (StubCodeDesc, StubCodeGenerator, StubCodeMark)
---
[Top](../index.html)

#### StubCodeGenerator クラス関連のクラス (StubCodeDesc, StubCodeGenerator, StubCodeMark)

これらは, 実行時のマシン語コード生成を行うためのクラス.
より具体的に言うと, stubcode に対する生成処理やデバッグ/出力処理等に関するクラス (See: [here](no7882z5r.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubCodeGenerator.hpp))
    // All the basic framework for stubcode generation/debugging/printing.
```


### クラス一覧(class list)

  * [StubCodeGenerator](#nogalTXRRz)
  * [StubCodeDesc](#no2paSRqD4)
  * [StubCodeMark](#nonrN3hITl)


---
## <a name="nogalTXRRz" id="nogalTXRRz">StubCodeGenerator</a>

### 概要(Summary)
何らかのスタブコードの生成処理で使用される一時オブジェクト(StackObjクラス) (の基底クラス).

実際にスタブコードを生成する機能を提供する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubCodeGenerator.hpp))
    // The base class for all stub-generating code generators.
    // Provides utility functions.
    
    class StubCodeGenerator: public StackObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

現状では以下のようなサブクラスが存在する.

* StubGenerator
* VM_Version_StubGenerator
* MethodHandlesAdapterGenerator
* ICacheStubGenerator

### 備考(Notes)
StubCodeGenerator に定義されている以下の 2つのメソッドは, 
補助クラスである StubCodeMark のコンストラクタ/デストラクタから呼び出される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubCodeGenerator.hpp))
      virtual void stub_prolog(StubCodeDesc* cdesc); // called by StubCodeMark constructor
      virtual void stub_epilog(StubCodeDesc* cdesc); // called by StubCodeMark destructor
```

デフォルトの挙動は以下の通り. 

* stub_prolog() は, デフォルトでは何もしない
* stub_epilog() は, 引数で渡された StubCodeDesc オブジェクトを内部のフィールドに記録するだけ.

これらは, 必要に応じてサブクラスでオーバーライドすることもできる.
ただし, 現状でオーバーライドしているのは Sparc 版の StubGenerator クラスのみ
(この場合はインストラクションキャッシュのキャッシュライン幅に合わせて先頭アドレスをアラインメントさせている).

#### 参考(for your information): StubCodeGenerator::stub_prolog()
See: [here](no1904-Lv.html) for details
#### 参考(for your information): StubCodeGenerator::stub_epilog()
See: [here](no1904LW1.html) for details
#### 参考(for your information): StubGenerator::stub_prolog()
See: [here](no17119w5c.html) for details



### 詳細(Details)
See: [here](../doxygen/classStubCodeGenerator.html) for details

---
## <a name="no2paSRqD4" id="no2paSRqD4">StubCodeDesc</a>

### 概要(Summary)
StubCodeGenerator が生成した stub の管理用のクラス.

StubCodeGenerator が生成した stub に関する「メタ情報」を格納している
(e.g. その stub の名前, 開始アドレス, 終端アドレス, etc).
この情報は主にデバッグや出力用途に使用される模様.


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubCodeGenerator.hpp))
    // A StubCodeDesc describes a piece of generated code (usually stubs).
    // This information is mainly useful for debugging and printing.
    // Currently, code descriptors are simply chained in a linked list,
    // this may have to change if searching becomes too slow.
    
    class StubCodeDesc: public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. コード中で StubCodeDesc 型の局所変数を宣言する.

   なお, 生成された StubCodeDesc オブジェクトは自動的に大域変数に登録される (内部構造も参照).

2. スタブ用のコードを生成する. 
   このコード生成の前後で StubCodeDesc::set_begin() 及び StubCodeDesc::set_end() を呼び出し, 
   コードの開始アドレスと終端アドレスを登録しておく.

2. 以下の static メソッドで, 生成済みの StubCodeDesc (あるいはその内部の情報) を取得できる.

   * StubCodeDesc::desc_for()
   * StubCodeDesc::desc_for_index()
   * StubCodeDesc::name_for()

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* StubCodeDesc クラスの _list フィールド (static フィールド)
  
  (正確には, このフィールドは StubCodeDesc の線形リストを格納するフィールド.
  StubCodeDesc オブジェクトは _next フィールドで次の StubCodeDesc オブジェクトを指せる構造になっている.
  生成した StubCodeDesc オブジェクトは全てこの線形リスト内に格納されている)


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubCodeGenerator.hpp))
      static StubCodeDesc* _list;                  // the list of all descriptors
      static int           _count;                 // length of list
```

* 各 StubCodeGenerator オブジェクトの _first_stub フィールド
* 各 StubCodeGenerator オブジェクトの _last_stub フィールド
  
  (正確には, これらのフィールドは StubCodeDesc の線形リスト(の先頭と最後)を格納するフィールド.
  StubCodeDesc オブジェクトは _next フィールドで次の StubCodeDesc オブジェクトを指せる構造になっている.
  その StubCodeGenerator 内で生成した StubCodeDesc オブジェクトは全てこの線形リスト内に格納されている)
  
  (格納している StubCodeDesc オブジェクト自体は, StubCodeDesc::_list が指しているものと重複)

#### 生成箇所(where its instances are created)
StubCodeMark::StubCodeMark() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
生成した StubCodeDesc オブジェクトは, 以下の箇所で(のみ)参照されている.

* (#TODO)
  
  * Relocation::runtime_address_to_index()

* relocInfo から Relocation への変換処理
  
  * Relocation::index_to_runtime_address()
  
* (#TODO)

  * CodeBlobCollector::collect()

* -XX:+PrintAssembly の処理

  * decode_env::print_address()

* トレース出力処理
  
  * frame::print_value_on()
  * frame::print_on_error()
  * os::print_location()
  * StubCodeGenerator::~StubCodeGenerator()


### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

  * _next
    
    線形リストを構成するためのポインタ. 次の StubCodeDesc を指す.

  * _group
    
    そのスタブコードが属しているグループ ("ICache", "StubRoutines", 等). 
    コンストラクタに渡した文字列がそのまま設定される.

  * _name
    
    そのスタブコードの名前 ("flush_icache_stub", "call_stub", 等).
    コンストラクタに渡した文字列がそのまま設定される.

  * _index

    そのスタブコードを指し示す一意な番号.
    現在は StubCodeDesc::_list 中での順番が格納されている.
    
  * _begin

    そのスタブコードの開始アドレス.
    
  * _end

    そのスタブコードの終端アドレス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubCodeGenerator.hpp))
      StubCodeDesc*        _next;                  // the next element in the linked list
      const char*          _group;                 // the group to which the stub code belongs
      const char*          _name;                  // the name assigned to the stub code
      int                  _index;                 // serial number assigned to the stub
      address              _begin;                 // points to the first byte of the stub code    (included)
      address              _end;                   // points to the first byte after the stub code (excluded)
```

なお, StubCodeDesc::_list につなぐ処理はコンストラクタで行われている.

See: [here](no19049fE.html) for details



### 詳細(Details)
See: [here](../doxygen/classStubCodeDesc.html) for details

---
## <a name="nonrN3hITl" id="nonrN3hITl">StubCodeMark</a>

### 概要(Summary)
StubCodeGenerator クラス用のユーティリティ・クラス.

StubCodeDesc オブジェクトの生成処理を簡単に行うための補助クラス(StackObjクラス). 
ソースコード上のスコープに連動して StubCodeDesc の生成処理(や初期化等)を行う.


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubCodeGenerator.hpp))
    // Stack-allocated helper class used to assciate a stub code with a name.
    // All stub code generating functions that use a StubCodeMark will be registered
    // in the global StubCodeDesc list and the generated stub code can be identified
    // later via an address pointing into it.
    
    class StubCodeMark: public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で StubCodeMark 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* 全プラットフォーム共通
  
  MethodHandlesAdapterGenerator::generate()

* Sparc の場合

  * ICacheStubGenerator::generate_icache_flush()
  * StubGenerator::generate_call_stub()
  * StubGenerator::generate_catch_exception()
  * StubGenerator::generate_forward_exception()
  * StubGenerator::generate_test_stop()
  * StubGenerator::generate_stop_subroutine()
  * StubGenerator::generate_flush_callers_register_windows()
  * StubGenerator::generate_atomic_xchg()
  * StubGenerator::generate_atomic_cmpxchg()
  * StubGenerator::generate_atomic_cmpxchg_long()
  * StubGenerator::generate_atomic_add()
  * StubGenerator::generate_handler_for_unsafe_access()
  * StubGenerator::generate_partial_subtype_check()
  * StubGenerator::generate_verify_oop_subroutine()
  * StubGenerator::generate_disjoint_byte_copy()
  * StubGenerator::generate_conjoint_byte_copy()
  * StubGenerator::generate_disjoint_short_copy()
  * StubGenerator::generate_fill()
  * StubGenerator::generate_conjoint_short_copy()
  * StubGenerator::generate_disjoint_int_copy()
  * StubGenerator::generate_conjoint_int_copy()
  * StubGenerator::generate_disjoint_long_copy()
  * StubGenerator::generate_conjoint_long_copy()
  * StubGenerator::generate_disjoint_oop_copy()
  * StubGenerator::generate_conjoint_oop_copy()
  * StubGenerator::generate_checkcast_copy()
  * StubGenerator::generate_unsafe_copy()
  * StubGenerator::generate_generic_copy()

* x86 (32/64 共通) の場合
  
  VM_Version_StubGenerator::generate_getPsrInfo()

* x86_32 の場合
  
  * ... 

* x86_64 の場合

  * ICacheStubGenerator::generate_icache_flush() 
  * StubGenerator::generate_call_stub()
  * StubGenerator::generate_catch_exception()
  * StubGenerator::generate_forward_exception()
  * StubGenerator::generate_atomic_xchg()
  * StubGenerator::generate_atomic_xchg_ptr()
  * StubGenerator::generate_atomic_cmpxchg()
  * StubGenerator::generate_atomic_cmpxchg_long()
  * StubGenerator::generate_atomic_add()
  * StubGenerator::generate_atomic_add_ptr()
  * StubGenerator::generate_orderaccess_fence()
  * StubGenerator::generate_get_previous_fp()
  * StubGenerator::generate_verify_mxcsr()
  * StubGenerator::generate_f2i_fixup()
  * StubGenerator::generate_f2l_fixup()
  * StubGenerator::generate_d2i_fixup()
  * StubGenerator::generate_d2l_fixup()
  * StubGenerator::generate_fp_mask()
  * StubGenerator::generate_handler_for_unsafe_access()
  * StubGenerator::generate_verify_oop()
  * StubGenerator::generate_disjoint_byte_copy()
  * StubGenerator::generate_conjoint_byte_copy()
  * StubGenerator::generate_disjoint_short_copy()
  * StubGenerator::generate_fill()
  * StubGenerator::generate_conjoint_short_copy()
  * StubGenerator::generate_disjoint_int_oop_copy()
  * StubGenerator::generate_conjoint_int_oop_copy()
  * StubGenerator::generate_disjoint_long_oop_copy()
  * StubGenerator::generate_conjoint_long_oop_copy()
  * StubGenerator::generate_checkcast_copy()
  * StubGenerator::generate_unsafe_copy()
  * StubGenerator::generate_generic_copy()
  * StubGenerator::generate_math_stubs()
  * StubGenerator::generate_math_stubs()
  * StubGenerator::generate_math_stubs()
  * StubGenerator::generate_math_stubs()
  * StubGenerator::generate_math_stubs()

### 内部構造(Internal structure)
コンストラクタで新しい StubCodeDesc を作成し, group や name, スタブコードの先頭アドレス等を設定する.

デストラクタでスタブコードの終端アドレスを設定する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/stubCodeGenerator.cpp))
    StubCodeMark::StubCodeMark(StubCodeGenerator* cgen, const char* group, const char* name) {
      _cgen  = cgen;
      _cdesc = new StubCodeDesc(group, name, _cgen->assembler()->pc());
      _cgen->stub_prolog(_cdesc);
      // define the stub's beginning (= entry point) to be after the prolog:
      _cdesc->set_begin(_cgen->assembler()->pc());
    }
    
    StubCodeMark::~StubCodeMark() {
      _cgen->assembler()->flush();
      _cdesc->set_end(_cgen->assembler()->pc());
      assert(StubCodeDesc::_list == _cdesc, "expected order on list");
      _cgen->stub_epilog(_cdesc);
      Forte::register_stub(_cdesc->name(), _cdesc->begin(), _cdesc->end());
    
      if (JvmtiExport::should_post_dynamic_code_generated()) {
        JvmtiExport::post_dynamic_code_generated(_cdesc->name(), _cdesc->begin(), _cdesc->end());
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classStubCodeMark.html) for details

---
