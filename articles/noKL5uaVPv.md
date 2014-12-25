---
layout: default
title: AbstractICache クラス関連のクラス (AbstractICache, ICacheStubGenerator)
---
[Top](../index.html)

#### AbstractICache クラス関連のクラス (AbstractICache, ICacheStubGenerator)

これらは, 実行時に生成したマシン語コードを補佐するためのクラス.
より具体的に言うと, マシン語を生成したメモリ領域に対して, インストラクションキャッシュを invalidate するためのクラス
(See: [here](no1904RLi.html) for details).


### クラス一覧(class list)

  * [AbstractICache](#noT3qUJUt8)
  * [ICacheStubGenerator](#no04l3dsJ3)


---
## <a name="noT3qUJUt8" id="noT3qUJUt8">AbstractICache</a>

### 概要(Summary)
インストラクションキャッシュを invalidate するための機能を納めた名前空間(AllStatic クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス (See: ICache).
また, インストラクションキャッシュの操作はプラットフォーム依存なので, 
サブクラスは cpu/ ディレクトリ下で定義されている
(といっても AllStatic なので abstract class というのも少し違和感があるが...).


```cpp
    ((cite: hotspot/src/share/vm/runtime/icache.hpp))
    // Interface for updating the instruction cache.  Whenever the VM modifies
    // code, part of the processor instruction cache potentially has to be flushed.
    
    // Default implementation is in icache.cpp, and can be hidden per-platform.
    // Most platforms must provide only ICacheStubGenerator::generate_icache_flush().
    // Platforms that don't require icache flushing can just nullify the public
    // members of AbstractICache in their ICache class.  AbstractICache should never
    // be referenced other than by deriving the ICache class from it.
    //
    // The code for the ICache class and for generate_icache_flush() must be in
    // architecture-specific files, i.e., icache_<arch>.hpp/.cpp
    
    class AbstractICache : AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classAbstractICache.html) for details

---
## <a name="no04l3dsJ3" id="no04l3dsJ3">ICacheStubGenerator</a>

### 概要(Summary)
AbstractICache クラス内で使用される補助クラス.

「指定された範囲の icache を invalidat するマシン語コード」を生成するための一時オブジェクト(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/icache.hpp))
    class ICacheStubGenerator : public StubCodeGenerator {
```

なお, (当然だが) このコードが生成された領域自体の invalidate には使えない.
なので, このコードは真っ先に作らないといけない
(このコードを生成した領域が既にキャッシュ済みだとどうしようもなくなる).


```cpp
    ((cite: hotspot/src/share/vm/runtime/icache.cpp))
      // Making this stub must be FIRST use of assembler
```

### 使われ方(Usage)
AbstractICache::initialize() 内で(のみ)使用されている.

### 内部構造(Internal structure)
内部には, (コンストラクタを除けば) 以下のメソッド(のみ)が定義されている.

  * ICacheStubGenerator::generate_icache_flush()

なお, ICacheStubGenerator::generate_icache_flush() を実装するときには以下のようにする必要がある.

       {
         StubCodeMark mark(this, "ICache", "flush_icache_stub");

         address start = __ pc();

         // emit flush stub asm code

         // Must be set here so StubCodeMark destructor can call the flush stub.
         *flush_icache_stub = (ICache::flush_icache_stub_t)start;
       };

StubCodeMark のデストラクタから AbstractICache::invalidate_range() が呼び出され, 
そこで「真っ先に _flush_icache_stub が作られたかどうかのチェック」が行われる (備考参照).

このため, 他のスタブのように
「関数内ではコードだけ作ってそのポインタを返し, 呼び元の方で flush_icache_stub にそのポインタを設定する」
というわけにはいかない
(この関数内で flush_icache_stub にポインタをセットしておく必要がある).


```cpp
    ((cite: hotspot/src/share/vm/runtime/icache.hpp))
      // Generate the icache flush stub.
      //
      // Since we cannot flush the cache when this stub is generated,
      // it must be generated first, and just to be sure, we do extra
      // work to allow a check that these instructions got executed.
      //
      // The flush stub has three parameters (see flush_icache_stub_t).
      //
      //   addr  - Start address, must be aligned at log2_line_size
      //   lines - Number of line_size icache lines to flush
      //   magic - Magic number copied to result register to make sure
      //           the stub executed properly
      //
      // A template for generate_icache_flush is
      //
      //    #define __ _masm->
      //
      //    void ICacheStubGenerator::generate_icache_flush(
      //      ICache::flush_icache_stub_t* flush_icache_stub
      //    ) {
      //      StubCodeMark mark(this, "ICache", "flush_icache_stub");
      //
      //      address start = __ pc();
      //
      //      // emit flush stub asm code
      //
      //      // Must be set here so StubCodeMark destructor can call the flush stub.
      //      *flush_icache_stub = (ICache::flush_icache_stub_t)start;
      //    };
      //
      //    #undef __
      //
      // The first use of flush_icache_stub must apply it to itself.  The
      // StubCodeMark destructor in generate_icache_flush will call Assembler::flush,
      // which in turn will call invalidate_range (see asm/assembler.cpp), which
      // in turn will call the flush stub *before* generate_icache_flush returns.
      // The usual method of having generate_icache_flush return the address of the
      // stub to its caller, which would then, e.g., store that address in
      // flush_icache_stub, won't work.  generate_icache_flush must itself set
      // flush_icache_stub to the address of the stub it generates before
      // the StubCodeMark destructor is invoked.
```

### 備考(Notes)
AbstractICache::invalidate_range() 内では, 
「真っ先に _flush_icache_stub が作られたかどうか」を (念のために) guarantee() でチェックしている.

#### 参考(for your information): AbstractICache::invalidate_range()
See: [here](no423066f.html) for details



### 詳細(Details)
See: [here](../doxygen/classICacheStubGenerator.html) for details

---
