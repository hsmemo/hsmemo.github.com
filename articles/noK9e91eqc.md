---
layout: default
title: Compile クラス関連のクラス (Compile, Compile::TracePhase, Compile::AliasType, Compile::Constant, Compile::ConstantTable, 及びそれらの補助クラス(CompileWrapper))
---
[Top](../index.html)

#### Compile クラス関連のクラス (Compile, Compile::TracePhase, Compile::AliasType, Compile::Constant, Compile::ConstantTable, 及びそれらの補助クラス(CompileWrapper))

これらは, C2 JIT Compiler 内の処理フェーズを表すクラス.
より具体的に言うと, JIT Compile 処理の全体を表すクラス.


### クラス一覧(class list)

  * [Compile](#noUijlhxGJ)
  * [Compile::TracePhase](#nolujL3A0W)
  * [Compile::AliasType](#no1-rbXkGU)
  * [Compile::ConstantTable](#nouFMbo1kx)
  * [Compile::Constant](#no9sPw-FzP)
  * [CompileWrapper](#nohFBkjQ0U)


---
## <a name="noUijlhxGJ" id="noUijlhxGJ">Compile</a>

### 概要(Summary)
Phase クラスの具象サブクラスの1つ.

JIT Compile 処理を実行する (= JIT Compile 作業全体のまとめ役).


```
    ((cite: hotspot/src/share/vm/opto/compile.hpp))
    //------------------------------Compile----------------------------------------
    // This class defines a top-level Compiler invocation.
    
    class Compile : public Phase {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* C2Compiler::compile_method()
* OptoRuntime::generate_stub()

### 内部構造(Internal structure)
2種類のコンストラクタを持つ.

* Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis )
  
  Java のメソッドの JIT コンパイル用 (OSR も含む).

* Compile::Compile( ciEnv* ci_env, TypeFunc_generator generator, address stub_function, const char *stub_name, int is_fancy_jump, bool pass_tls, bool save_arg_registers, bool return_pc )

  HotSpot が内部的に使用する runtime stub の JIT コンパイル用.


```
    ((cite: hotspot/src/share/vm/opto/compile.hpp))
      // Major entry point.  Given a Scope, compile the associated method.
      // For normal compilations, entry_bci is InvocationEntryBci.  For on stack
      // replacement, entry_bci indicates the bytecode for which to compile a
      // continuation.
      Compile(ciEnv* ci_env, C2Compiler* compiler, ciMethod* target,
              int entry_bci, bool subsume_loads, bool do_escape_analysis);
    
      // Second major entry point.  From the TypeFunc signature, generate code
      // to pass arguments from the Java calling convention to the C calling
      // convention.
      Compile(ciEnv* ci_env, const TypeFunc *(*gen)(),
              address stub_function, const char *stub_name,
              int is_fancy_jump, bool pass_tls,
              bool save_arg_registers, bool return_pc);
```




### 詳細(Details)
See: [here](../doxygen/classCompile.html) for details

---
## <a name="nolujL3A0W" id="nolujL3A0W">Compile::TracePhase</a>

### 概要(Summary)
Compile クラス内で使用される補助クラス.

特殊な TraceTime クラス.
時間の記録処理に加えて, Compile クラスが保持している CompileLog に開始時／終了時に出力する機能を備える.

(なお, このクラスによる時間の記録処理は TimeCompiler オプションがセットされている場合にのみ行われる)


```
    ((cite: hotspot/src/share/vm/opto/compile.hpp))
      // Variant of TraceTime(NULL, &_t_accumulator, TimeCompiler);
      // Integrated with logging.  If logging is turned on, and dolog is true,
      // then brackets are put into the log, with time stamps and node counts.
      // (The time collection itself is always conditionalized on TimeCompiler.)
      class TracePhase : public TraceTime {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で Compile::TracePhase 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
C2 JIT Compiler 関連の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
コンストラクタとデストラクタで Compile クラスが保持している CompileLog に出力を行っている.


```
    ((cite: hotspot/src/share/vm/opto/compile.cpp))
    Compile::TracePhase::TracePhase(const char* name, elapsedTimer* accumulator, bool dolog)
      : TraceTime(NULL, accumulator, false NOT_PRODUCT( || TimeCompiler ), false)
    {
      if (dolog) {
        C = Compile::current();
        _log = C->log();
      } else {
        C = NULL;
        _log = NULL;
      }
      if (_log != NULL) {
        _log->begin_head("phase name='%s' nodes='%d'", name, C->unique());
        _log->stamp();
        _log->end_head();
      }
    }
    
    Compile::TracePhase::~TracePhase() {
      if (_log != NULL) {
        _log->done("phase nodes='%d'", C->unique());
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classCompile_1_1TracePhase.html) for details

---
## <a name="no1-rbXkGU" id="no1-rbXkGU">Compile::AliasType</a>

### 概要(Summary)
Compile クラス内で使用される補助クラス.

ポインタのエイリアス解析情報を管理するためのクラス.
1つの Compile::AliasType オブジェクトが 1つの「エイリアスになり得るポインタの集合」に対応する.

なお, エイリアス解析情報の管理は MergeMemNode と連携して行われている.
具体的に言うと, Compile::AliasType オブジェクトは Compile::_alias_types フィールドの配列に収められている.
そして MergeMemNode は (Compile::AliasType オブジェクト自体を指す代わりに) 該当する添字番号(index)を格納している.


```
    ((cite: hotspot/src/share/vm/opto/compile.hpp))
      // Information per category of alias (memory slice)
      class AliasType {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Compile オブジェクトの _alias_types フィールドに(のみ)格納されている.

(正確には, このフィールドは Compile::AliasType のポインタの配列を格納するフィールド.
この中に, その Compile オブジェクト内で生成された全ての Compile::AliasType オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
配列用のメモリ領域は以下の箇所で(のみ)確保されている. 

* Compile::Init()
  
  Compile::_alias_types フィールドの初期化

* Compile::grow_alias_types()
  
  Compile::_alias_types フィールドの realloc

そのメモリ領域中に個別の Compile::AliasType オブジェクトを書き込む作業は 
Compile::AliasType::Init() で(のみ)行われている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* Compile::Init()
  -> Compile::AliasType::Init()

* Compile::find_alias_type()
  -> Compile::AliasType::Init()
```

### 内部構造(Internal structure)
以下の 3種類がデフォルトで用意されている
(その他の Compile::AliasType オブジェクトは, 必要に応じて Compile::find_alias_type() で生成される).


```
    ((cite: hotspot/src/share/vm/opto/compile.hpp))
      // Fixed alias indexes.  (See also MergeMemNode.)
      enum {
        AliasIdxTop = 1,  // pseudo-index, aliases to nothing (used as sentinel value)
        AliasIdxBot = 2,  // pseudo-index, aliases to everything
        AliasIdxRaw = 3   // hard-wired index for TypeRawPtr::BOTTOM
      };
```





### 詳細(Details)
See: [here](../doxygen/classCompile_1_1AliasType.html) for details

---
## <a name="nouFMbo1kx" id="nouFMbo1kx">Compile::ConstantTable</a>

### 概要(Summary)
Compile クラス内で使用される補助クラス.

即値としては表せない定数値をまとめて管理するためのテーブル
(e.g. 即値フィールドに収まらない整数値, 浮動小数点の即値, lookupswitch/tableswitch 用の飛び先テーブル, etc).

使用する際には JIT コンパイル中に定数値を登録しておき, 
JIT コンパイルの終了時にテーブルそのものを CodeBuffer に出力する.
実行時にはそこからロードすることになる.


```
    ((cite: hotspot/src/share/vm/opto/compile.hpp))
      // Constant table.
      class ConstantTable {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. Compile::ConstantTable::add(Constant& con) で定数値(Compile::Constantオブジェクト)を登録する.

   (なお, このメソッドを呼び出すコードは ADLC によって生成されている.
   MachConstantNode::eval_constant() (を各サブクラスがオーバーライドしたもの) を参照.
   なお Compile::ConstantTable オブジェクトは AD ファイル中では $constanttablebase や $constanttablebase で参照されている)

1. Compile::ConstantTable::allocate_jump_table() で lookupswitch/tableswitch 用の飛び先テーブルを登録する

   (なお, このメソッドを呼び出すコードは ADLC によって生成されている.
   MachConstantNode::eval_constant() (を各サブクラスがオーバーライドしたもの) を参照.
   なお Compile::ConstantTable オブジェクトは AD ファイル中では $constanttablebase や $constanttablebase で参照されている)

2. Compile::ConstantTable::calculate_offsets_and_size() で, 
   それぞれの定数値の位置(オフセット)やテーブル全体の大きさを確定させる.

3. Compile::ConstantTable::emit() で, 登録した定数値が CodeBuffer に出力される.

3. Compile::ConstantTable::fill_jump_table() で, 
   登録した lookupswitch/tableswitch 用の飛び先テーブルが CodeBuffer に出力される
   (なお出力時に relocation も行われる).

4. Compile::ConstantTable::find_offset() で, 登録した値の位置(オフセット)を取得できる.

#### インスタンスの格納場所(where its instances are stored)
各 Compile オブジェクトの _constant_table フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(Compile クラスの _constant_table フィールドは, ポインタ型ではなく実体なので,
 Compile オブジェクトの生成時に一緒に生成される)





### 詳細(Details)
See: [here](../doxygen/classCompile_1_1ConstantTable.html) for details

---
## <a name="no9sPw-FzP" id="no9sPw-FzP">Compile::Constant</a>

### 概要(Summary)
Compile クラス用の補助クラス.

Compile::ConstantTable に登録されている定数値を表すクラス.
1つの Compile::Constant オブジェクトが 1つの定数値に対応する.

(なお, このクラスはValueObjクラスではないが, 扱われ方はValueObjクラスに近い)


```
    ((cite: hotspot/src/share/vm/opto/compile.hpp))
      // Constant entry of the constant table.
      class Constant {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 Compile::ConstantTable オブジェクトの _constants フィールド
  
  (正確には, このフィールドは Compile::Constant の GrowableArray を格納するフィールド.
  この中に, その Compile::ConstantTable 用の全ての Compile::Constant オブジェクトが格納されている)

* 各 MachConstantNode オブジェクトの _constant フィールド
  
  その MachConstantNode が表す定数値

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* (MachConstantNode クラスの _constant フィールドは, ポインタ型ではなく実体なので,
  MachConstantNode オブジェクトの生成時に一緒に生成される)

* Compile::ConstantTable::add(BasicType type, jvalue value)

* Compile::ConstantTable::allocate_jump_table()

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(そして, メソッドはこれらのフィールドへのアクセサメソッドのみ).


```
    ((cite: hotspot/src/share/vm/opto/compile.hpp))
        BasicType _type;
        jvalue    _value;
        int       _offset;         // offset of this constant (in bytes) relative to the constant table base.
        bool      _can_be_reused;  // true (default) if the value can be shared with other users.
```





### 詳細(Details)
See: [here](../doxygen/classCompile_1_1Constant.html) for details

---
## <a name="nohFBkjQ0U" id="nohFBkjQ0U">CompileWrapper</a>

### 概要(Summary)
Compile クラス内で使用される補助クラス.

Compile クラスによる JIT コンパイル処理を簡単に記述するための補助クラス(StackObjクラス).
Compile オブジェクトの初期化や後始末をソースコード上のスコープに合わせて自動で行ってくれる.


```
    ((cite: hotspot/src/share/vm/opto/compile.cpp))
    // ============================================================================
    //------------------------------CompileWrapper---------------------------------
    class CompileWrapper : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis )
* Compile::Compile( ciEnv* ci_env, TypeFunc_generator generator, address stub_function, const char *stub_name, int is_fancy_jump, bool pass_tls, bool save_arg_registers, bool return_pc )

### 内部構造(Internal structure)
コンストラクタで初期化処理を行い, デストラクタで後始末を行う.


```
    ((cite: hotspot/src/share/vm/opto/compile.cpp))
    CompileWrapper::CompileWrapper(Compile* compile) : _compile(compile) {
      // the Compile* pointer is stored in the current ciEnv:
      ciEnv* env = compile->env();
      assert(env == ciEnv::current(), "must already be a ciEnv active");
      assert(env->compiler_data() == NULL, "compile already active?");
      env->set_compiler_data(compile);
      assert(compile == Compile::current(), "sanity");
    
      compile->set_type_dict(NULL);
      compile->set_type_hwm(NULL);
      compile->set_type_last_size(0);
      compile->set_last_tf(NULL, NULL);
      compile->set_indexSet_arena(NULL);
      compile->set_indexSet_free_block_list(NULL);
      compile->init_type_arena();
      Type::Initialize(compile);
      _compile->set_scratch_buffer_blob(NULL);
      _compile->begin_method();
    }
    CompileWrapper::~CompileWrapper() {
      _compile->end_method();
      if (_compile->scratch_buffer_blob() != NULL)
        BufferBlob::free(_compile->scratch_buffer_blob());
      _compile->env()->set_compiler_data(NULL);
    }
```




### 詳細(Details)
See: [here](../doxygen/classCompileWrapper.html) for details

---
