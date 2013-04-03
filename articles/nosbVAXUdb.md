---
layout: default
title: CodeBuffer クラス関連のクラス (CodeOffsets, CodeSection, CodeComments, CodeBuffer, 及びそれらの補助クラス(CodeComment))
---
[Top](../index.html)

#### CodeBuffer クラス関連のクラス (CodeOffsets, CodeSection, CodeComments, CodeBuffer, 及びそれらの補助クラス(CodeComment))

これらは, 動的コード生成を補佐するクラス.
より具体的に言うと, Assembler クラスが書き出したマシン語(および, 再配置情報, コメント情報, 等)を管理するためのクラス
(See: [here](no7882z5r.html) for details).


### クラス一覧(class list)

  * [CodeBuffer](#no1WtsOx9P)
  * [CodeOffsets](#noojW6bQwl)
  * [CodeSection](#nosPp0WSBF)
  * [CodeComments](#nomYVeTC-W)
  * [CodeComment](#norzP1Y8qU)


---
## <a name="no1WtsOx9P" id="no1WtsOx9P">CodeBuffer</a>

### 概要(Summary)
Assembler クラスがマシン語を出力するためのメモリ領域を表すクラス.

(正確には, CodeBuffer 自体は作業用の一時オブジェクト(StackObjクラス).
Assembler は CodeBuffer にマシン語を書き出すが, 
書き出された内容は最終的には nmethod や CodeBlob にコピーされて使用される)

内部には複数の CodeSection を持ち,
それらの並行で埋めていって, 最後に 1つにまとめている模様.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
    // A CodeBuffer describes a memory space into which assembly
    // code is generated.  This memory space usually occupies the
    // interior of a single BufferBlob, but in some cases it may be
    // an arbitrary span of memory, even outside the code cache.
    //
    // A code buffer comes in two variants:
    //
    // (1) A CodeBuffer referring to an already allocated piece of memory:
    //     This is used to direct 'static' code generation (e.g. for interpreter
    //     or stubroutine generation, etc.).  This code comes with NO relocation
    //     information.
    //
    // (2) A CodeBuffer referring to a piece of memory allocated when the
    //     CodeBuffer is allocated.  This is used for nmethod generation.
    //
    // The memory can be divided up into several parts called sections.
    // Each section independently accumulates code (or data) an relocations.
    // Sections can grow (at the expense of a reallocation of the BufferBlob
    // and recopying of all active sections).  When the buffered code is finally
    // written to an nmethod (or other CodeBlob), the contents (code, data,
    // and relocations) of the sections are padded to an alignment and concatenated.
    // Instructions and data in one section can contain relocatable references to
    // addresses in a sibling section.
    
    class CodeBuffer: public StackObj {
```

### 内部構造(Internal structure)
内部には 3つの CodeSection オブジェクトを持つ (See: CodeSection). 
最終的に CodeCache 内にコピーする際には, それぞれの CodeSection 中の空の部分は詰めてコピーされる模様.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
      CodeSection  _consts;             // constants, jump tables
      CodeSection  _insts;              // instructions (the main section)
      CodeSection  _stubs;              // stubs (call site support), deopt, exception handling
```

```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.cpp))
    // The structure of the CodeBuffer while code is being accumulated:
    //
    //    _total_start ->    \
    //    _insts._start ->              +----------------+
    //                                  |                |
    //                                  |     Code       |
    //                                  |                |
    //    _stubs._start ->              |----------------|
    //                                  |                |
    //                                  |    Stubs       | (also handlers for deopt/exception)
    //                                  |                |
    //    _consts._start ->             |----------------|
    //                                  |                |
    //                                  |   Constants    |
    //                                  |                |
    //                                  +----------------+
    //    + _total_size ->              |                |
    //
    // When the code and relocations are copied to the code cache,
    // the empty parts of each section are removed, and everything
    // is copied into contiguous locations.
```



### 詳細(Details)
See: [here](../doxygen/classCodeBuffer.html) for details

---
## <a name="noojW6bQwl" id="noojW6bQwl">CodeOffsets</a>

### 概要(Summary)
Assembler クラスによるコード生成作業中に, 
生成したコードのエントリポイントのアドレスを覚えておくための一時オブジェクト(StackObj クラス).

(主に JIT 生成コード用?? #TODO)


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
    class CodeOffsets: public StackObj {
```

### 使われ方(Usage)
JIT コンパイラによって収集されたエントリポイントの情報が, 
CodeOffsets という形で nmethod のコンストラクタに引き渡されたりしている.


```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    nmethod::nmethod(
    ...
      CodeOffsets* offsets,
    ...
      )
    ...
    {
    ...
        // Exception handler and deopt handler are in the stub section
        assert(offsets->value(CodeOffsets::Exceptions) != -1, "must be set");
        assert(offsets->value(CodeOffsets::Deopt     ) != -1, "must be set");
        _exception_offset        = _stub_offset          + offsets->value(CodeOffsets::Exceptions);
        _deoptimize_offset       = _stub_offset          + offsets->value(CodeOffsets::Deopt);
        if (offsets->value(CodeOffsets::DeoptMH) != -1) {
          _deoptimize_mh_offset  = _stub_offset          + offsets->value(CodeOffsets::DeoptMH);
    ...
        if (offsets->value(CodeOffsets::UnwindHandler) != -1) {
          _unwind_handler_offset = code_offset()         + offsets->value(CodeOffsets::UnwindHandler);
    ...
        _entry_point             = code_begin()          + offsets->value(CodeOffsets::Entry);
        _verified_entry_point    = code_begin()          + offsets->value(CodeOffsets::Verified_Entry);
        _osr_entry_point         = code_begin()          + offsets->value(CodeOffsets::OSR_Entry);
    ...
    }
```

### 内部構造(Internal structure)
エントリポイントは一つではなく, 以下のように複数の種類が存在する.
(それぞれどういうエントリポイントか? #TODO)


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
      enum Entries { Entry,
                     Verified_Entry,
                     Frame_Complete, // Offset in the code where the frame setup is (for forte stackwalks) is complete
                     OSR_Entry,
                     Dtrace_trap = OSR_Entry,  // dtrace probes can never have an OSR entry so reuse it
                     Exceptions,     // Offset where exception handler lives
                     Deopt,          // Offset where deopt handler lives
                     DeoptMH,        // Offset where MethodHandle deopt handler lives
                     UnwindHandler,  // Offset to default unwind handler
                     max_Entries };
```



### 詳細(Details)
See: [here](../doxygen/classCodeOffsets.html) for details

---
## <a name="nosPp0WSBF" id="nosPp0WSBF">CodeSection</a>

### 概要(Summary)
実際のマシン語コード列, 及び再配置(relocation)情報を格納するためのクラス.

CodeBuffer 内に格納されている (See: CodeBuffer).
これらの CodeSection が並行に埋められていき, 最後にそれらの内容が1つにまとめられる.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
    // This class represents a stream of code and associated relocations.
    // There are a few in each CodeBuffer.
    // They are filled concurrently, and concatenated at the end.
    class CodeSection VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
CodeSection 内では,
_start 〜 _limit というアドレス範囲にマシン語列が入っている模様 (なお, _end までが使用済, それ以降が未使用スペース).

また、 _locs_start 〜 _locs_limit というアドレス範囲に relocInfo オブジェクトの配列が納められており, 
ここに relocation 情報を格納している模様
(なお, _locs_end までが使用済, それ以降が未使用スペース) (See: relocInfo).


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.cpp))
    // The structure of a CodeSection:
    //
    //    _start ->           +----------------+
    //                        | machine code...|
    //    _end ->             |----------------|
    //                        |                |
    //                        |    (empty)     |
    //                        |                |
    //                        |                |
    //                        +----------------+
    //    _limit ->           |                |
    //
    //    _locs_start ->      +----------------+
    //                        |reloc records...|
    //                        |----------------|
    //    _locs_end ->        |                |
    //                        |                |
    //                        |    (empty)     |
    //                        |                |
    //                        |                |
    //                        +----------------+
    //    _locs_limit ->      |                |
    // The _end (resp. _limit) pointer refers to the first
    // unused (resp. unallocated) byte.
```



### 詳細(Details)
See: [here](../doxygen/classCodeSection.html) for details

---
## <a name="nomYVeTC-W" id="nomYVeTC-W">CodeComments</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時以外には空のクラスとして定義される).

block_comment 機能を実現するためのクラス (See: [here](no7882AEy.html) for details).


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
    class CodeComments VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 CodeBuffer オブジェクトの _comments フィールド
* 各 CodeBlob オブジェクトの _comments フィールド


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
      CodeComments _comments;
```


```
    ((cite: hotspot/src/share/vm/code/codeBlob.hpp))
      CodeComments _comments;
```

### 内部構造(Internal structure)
block_comment() が呼ばれた際の offset と文字列を内部に記録している.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.cpp))
    void CodeBuffer::block_comment(intptr_t offset, const char * comment) {
      _comments.add_comment(offset, comment);
    }
```

そして, コード完成後に呼ばれる CodeBuffer::copy_code_to() によって, 
CodeBuffer 内の CodeComments に溜められたコメントが CodeBlob にコピーされる.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.cpp))
    void CodeBuffer::copy_code_to(CodeBlob* dest_blob) {
    ...
      // transfer comments from buffer to blob
      dest_blob->set_comments(_comments);
```

なお, 開発時用のクラスであるため, #ifdef PRODUCT 時には全てのメソッドの中身が空になる
(PRODUCT_RETURN は #ifdef PRODUCT 時には '{}' に展開される. (See: PRODUCT_RETURN)).
また #ifdef PRODUCT 時にはフィールドも定義されない.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
      void add_comment(intptr_t offset, const char * comment) PRODUCT_RETURN;
      void print_block_comment(outputStream* stream, intptr_t offset)  PRODUCT_RETURN;
      void assign(CodeComments& other)  PRODUCT_RETURN;
      void free() PRODUCT_RETURN;
```


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
    #ifndef PRODUCT
      CodeComment* _comments;
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classCodeComments.html) for details

---
## <a name="norzP1Y8qU" id="norzP1Y8qU">CodeComment</a>

CodeComments クラス内で使われる補助クラス.
実際のコメント情報を溜めておくための線形リスト.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.cpp))
    class CodeComment: public CHeapObj {
```

### 内部構造(Internal structure)
以下のようなフィールドからなる
(要は _next でつながっていく線形リストになっている).


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.cpp))
      intptr_t     _offset;
      const char * _comment;
      CodeComment* _next;
```

なお, 文字列部分(_comment フィールド)は, ちゃんと strdup() でメモリを新規に確保している.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.cpp))
      CodeComment(intptr_t offset, const char * comment) {
        _offset = offset;
        _comment = os::strdup(comment);
        _next = NULL;
      }
```

  (このため, デストラクタ内で free している)

```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.cpp))
      ~CodeComment() {
        assert(_next == NULL, "wrong interface for freeing list");
        os::free((void*)_comment);
      }
```




### 詳細(Details)
See: [here](../doxygen/classCodeComment.html) for details

---
