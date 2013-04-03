---
layout: default
title: Disassembler クラス (Disassembler, 及びその補助クラス(decode_env))
---
[Top](../index.html)

#### Disassembler クラス (Disassembler, 及びその補助クラス(decode_env))

これらは, JIT コンパイラに対するデバッグ用のクラス. 生成したマシン語を逆アセンブルして表示する為に使われる.



### クラス一覧(class list)

  * [Disassembler](#noK0HoVz_i)
  * [decode_env](#noTv8vHI32)


---
## <a name="noK0HoVz_i" id="noK0HoVz_i">Disassembler</a>

### 概要(Summary)
トラブルシューティング用/デバッグ用(開発時用)のクラス?? (関連する diagnostic オプションまたは develop オプション/notproduct オプションが指定されている場合にのみ使用される?? (<= 通常時でも使用されるパスはある?? #TODO)) (See: PrintAdapterHandlers, PrintAssembly, PrintInterpreter, PrintOptoAssembly, PrintSignatureHandlers, PrintStubCode, TracePatching, G1SATBPrintStubs, Verbose, WizardMode, )

生成したマシン語を逆アセンブルして表示するための機能を納めた名前空間
(このクラスは AllStatic ではないが, static なフィールド／メソッドしか持たない).


```
    ((cite: hotspot/src/share/vm/compiler/disassembler.hpp))
    // The disassembler prints out assembly code annotated
    // with Java specific information.
    
    class Disassembler {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
Disassembler::decode() を呼び出すと, 逆アセンブル結果が出力される.

なお, Disassembler::decode() は 3種類存在している.


```
    ((cite: hotspot/src/share/vm/compiler/disassembler.hpp))
      static void decode(CodeBlob *cb,               outputStream* st = NULL);
      static void decode(nmethod* nm,                outputStream* st = NULL);
      static void decode(address begin, address end, outputStream* st = NULL);
```

#### このクラスの使用箇所(where this class is used)
以下の箇所で(のみ)使用されている.

* CodeSection::decode()
* CodeBuffer::decode()
* RuntimeStub::print_on()
* SingletonBlob::print_on()
* nmethod::print_code()
* disnm()
* Runtime1::patch_code() (TracePatching オプション指定時のみ)
* generate_satb_log_enqueue_if_necessary() (sparc の場合)  (G1SATBPrintStubs オプション指定時のみ)
* generate_dirty_card_log_enqueue_if_necessary() (sparc の場合)  (G1SATBPrintStubs オプション指定時のみ)
* os::find() (linux の場合) (Verbose オプション指定時のみ)
* os::find() (solaris の場合) (Verbose オプション指定時のみ)
* CodeBlob::trace_new_stub() (PrintStubCode オプション指定時のみ)
* nmethod::new_native_nmethod() (PrintAssembly オプション指定時のみ)
* nmethod::new_dtrace_nmethod() (PrintAssembly オプション指定時のみ)
* nmethod::new_nmethod() (PrintAssembly オプション指定時のみ)
* VtableStubs::create_stub() (PrintAdapterHandlers オプション指定時のみ)
* InterpreterCodelet::print_on() (PrintInterpreter オプション指定時のみ)
* SignatureHandlerLibrary::add() (PrintSignatureHandlers オプション指定時のみ)
* Compile::Compile() (PrintOptoAssembly オプション指定時, もしくは CompilerOracle の設定で対象メソッドに PrintOptoAssembly オプションが設定されている場合)
* frame::print_value_on() (NOT_PRODUCT && WizardMode オプション指定時 && Verbose オプション指定時)
* AdapterHandlerLibrary::get_adapter() (#ifndef PRODUCT && PrintAdapterHandlers オプション指定時)
* StubCodeGenerator::~StubCodeGenerator() (PrintStubCode オプション指定時)

### 内部構造(Internal structure)
内部では hsdis-$LIBARCH というライブラリ (e.g. libhsdis-amd64.so) の decode_instruction() 関数を使用している.
このライブラリは hotspot/src/share/tools/hsdis をコンパイルすると作られる (See: [here](no7882BMt.html) for details).


```
    ((cite: hotspot/src/share/vm/compiler/disassembler.cpp))
    static const char hsdis_library_name[] = "hsdis-"HOTSPOT_LIB_ARCH;
    static const char decode_instructions_name[] = "decode_instructions";
```




### 詳細(Details)
See: [here](../doxygen/classDisassembler.html) for details

---
## <a name="noTv8vHI32" id="noTv8vHI32">decode_env</a>

### 概要(Summary)
Disassembler クラス内で使用される補助クラス.
逆アセンブル作業中に使われる一時オブジェクト (なお, このクラスは StackObjクラスではないが現状では局所変数としてのみ生成されている).

このクラスがマシン依存な箇所を担当している
(というか Disassembler クラスは実際の処理はほとんどせず decode_env クラスに丸投げしているだけ).


```
    ((cite: hotspot/src/share/vm/compiler/disassembler.cpp))
    class decode_env {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* Disassembler::decode(CodeBlob* cb, outputStream* st)
* Disassembler::decode(address start, address end, outputStream* st)
* Disassembler::decode(nmethod* nm, outputStream* st)

### 内部構造(Internal structure)
実際の逆アセンブル処理自体は hsdis ライブラリが行っている.

ただし, hsdis ライブラリから (event_to_env や printf_to_env という関数を介して) 
decode_env クラスのメソッドがコールバックで呼び出されている模様.

(以下の this がコールバックに引数として渡される decode_env オブジェクト)


```
    ((cite: hotspot/src/share/vm/compiler/disassembler.cpp))
    address decode_env::decode_instructions(address start, address end) {
    ...
      return (address)
        (*Disassembler::_decode_instructions)(start, end,
                                              &event_to_env,  (void*) this,
                                              &printf_to_env, (void*) this,
                                              options());
```

### 備考(Notes)
このクラスの挙動は diagnostic オプションである PrintAssemblyOptions で設定できる模様.


```
    ((cite: hotspot/src/share/vm/compiler/disassembler.cpp))
    decode_env::decode_env(CodeBlob* code, outputStream* output) {
    ...
      collect_options(PrintAssemblyOptions);
    
      if (strstr(options(), "hsdis-")) {
        if (strstr(options(), "hsdis-print-raw"))
          _print_raw = (strstr(options(), "xml") ? 2 : 1);
        if (strstr(options(), "hsdis-print-pc"))
          _print_pc = !_print_pc;
        if (strstr(options(), "hsdis-print-bytes"))
          _print_bytes = !_print_bytes;
      }
      if (strstr(options(), "help")) {
        tty->print_cr("PrintAssemblyOptions help:");
        tty->print_cr("  hsdis-print-raw       test plugin by requesting raw output");
        tty->print_cr("  hsdis-print-raw-xml   test plugin by requesting raw xml");
        tty->print_cr("  hsdis-print-pc        turn off PC printing (on by default)");
        tty->print_cr("  hsdis-print-bytes     turn on instruction byte output");
        tty->print_cr("combined options: %s", options());
      }
```




### 詳細(Details)
See: [here](../doxygen/classdecode__env.html) for details

---
