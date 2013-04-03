---
layout: default
title: CompressedStream クラス関連のクラス (CompressedStream, CompressedReadStream, CompressedWriteStream)
---
[Top](../index.html)

#### CompressedStream クラス関連のクラス (CompressedStream, CompressedReadStream, CompressedWriteStream)

これらは, 何らかのデータを圧縮しながら書き込む(あるいは逆に圧縮されたデータを解凍しながら読み込む)ためのクラス.

JIT コンパイル中にデバッグ情報を蓄積していく際に使われている模様 (他の所でも使われている?? #TODO)

(行う圧縮処理としては, 例えば, 符号の有り無しが分かっているデータについて最初の符号bitを削る, 等)



### クラス一覧(class list)

  * [CompressedStream](#noWlVLeYZM)
  * [CompressedReadStream](#nojV7_zcTT)
  * [CompressedWriteStream](#nowSC_1g43)


---
## <a name="noWlVLeYZM" id="noWlVLeYZM">CompressedStream</a>

### 概要(Summary)
CompressedStream 関連のクラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/code/compressedStream.hpp))
    // Simple interface for filing out and filing in basic types
    // Used for writing out and reading in debugging information.
    
    class CompressedStream : public ResourceObj {
```

### 備考(Notes)
このファイル以外にもサブクラスが存在するので注意.
(hotspot/src/share/vm/code/debugInfo.hpp や hotspot/src/share/vm/oops/methodOop.hpp も参照)

  * CompressedStream
    * CompressedReadStream
      * DebugInfoReadStream
      * CompressedLineNumberReadStream
    * CompressedWriteStream
      * DebugInfoWriteStream
      * CompressedLineNumberWriteStream



### 詳細(Details)
See: [here](../doxygen/classCompressedStream.html) for details

---
## <a name="nojV7_zcTT" id="nojV7_zcTT">CompressedReadStream</a>

### 概要(Summary)
CompressedStream クラスのサブクラスの1つ.

圧縮されたデータを解凍しながら読み込む為のメソッドが定義されている.


```
    ((cite: hotspot/src/share/vm/code/compressedStream.hpp))
    class CompressedReadStream : public CompressedStream {
```

### 使われ方(Usage)
例えば, OopMapStream 内で使われている.


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
    class OopMapStream : public StackObj {
    ...
      CompressedReadStream* _stream;
```

また, DepStream 内でも使われている.


```
    ((cite: hotspot/src/share/vm/code/dependencies.hpp))
      class DepStream {
    ...
        CompressedReadStream  _bytes;
```



### 詳細(Details)
See: [here](../doxygen/classCompressedReadStream.html) for details

---
## <a name="nowSC_1g43" id="nowSC_1g43">CompressedWriteStream</a>

### 概要(Summary)
CompressedStream クラスのサブクラスの1つ.

データを圧縮しながら書き込む為のメソッドが定義されている.


```
    ((cite: hotspot/src/share/vm/code/compressedStream.hpp))
    class CompressedWriteStream : public CompressedStream {
```

### 使われ方(Usage)
例えば, OopMap の中で使われている.


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.cpp))
    OopMap::OopMap(int frame_size, int arg_count) {
    ...
      set_write_stream(new CompressedWriteStream(32));
```

また, Dependencies 内でも使われている.


```
    ((cite: hotspot/src/share/vm/code/dependencies.cpp))
    void Dependencies::encode_content_bytes() {
    ...
      // cast is safe, no deps can overflow INT_MAX
      CompressedWriteStream bytes((int)estimate_size_in_bytes());
```




### 詳細(Details)
See: [here](../doxygen/classCompressedWriteStream.html) for details

---
