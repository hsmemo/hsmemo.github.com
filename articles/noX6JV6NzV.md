---
layout: default
title: outputStream クラス関連のクラス (outputStream, ttyLocker, ttyUnlocker, stringStream, fileStream, fdStream, staticBufferStream, bufferedStream, networkStream)
---
[Top](../index.html)

#### outputStream クラス関連のクラス (outputStream, ttyLocker, ttyUnlocker, stringStream, fileStream, fdStream, staticBufferStream, bufferedStream, networkStream)

これらは, HotSpot 内での出力処理を担当するユーティリティ・クラス.

(要するに C++ の ostream クラスや Java の java.io.OutputStream クラスのようなもの.
 ファイルへの出力やメモリ上での文字列の構築処理に使用される)

これらのクラスは以下のような継承関係を持つ.

* outputStream : 基底クラス
    * stringStream
    * fileStream
    * fdStream
    * staticBufferStream
    * bufferedStream
        * networkStream
    * xmlTextStream
        * defaultStream
    * xmlStream
        * CompileLog


### クラス一覧(class list)

  * [outputStream](#noT-cvVHwh)
  * [ttyLocker](#noojgmro7p)
  * [ttyUnlocker](#nohYjCc66_)
  * [stringStream](#notihfGXvW)
  * [fileStream](#noyIkgTkBM)
  * [fdStream](#noar1m65Qo)
  * [staticBufferStream](#nokzolv4K2)
  * [bufferedStream](#norq651IBm)
  * [networkStream](#noYrBr5Cn3)


---
## <a name="noT-cvVHwh" id="noT-cvVHwh">outputStream</a>

### 概要(Summary)
全ての outputStream クラスの基底クラス.

なおコメントによると, 
「tty という outputStream 型の大域変数を1つ用意しているので
(特に問題が無ければ) 出力処理は tty->print() や tty->print_cr() で行って欲しい」
とのこと.

どうしても tty が使えない場合(初期化の早い段階やオーバーヘッドが大きい場合)には
jio_fprintf(defaultStream::output_stream(), "...") で出力すればいいらしい.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    // Output streams for printing
    //
    // Printing guidelines:
    // Where possible, please use tty->print() and tty->print_cr().
    // For product mode VM warnings use warning() which internally uses tty.
    // In places where tty is not initialized yet or too much overhead,
    // we may use jio_printf:
    //     jio_fprintf(defaultStream::output_stream(), "Message");
    // This allows for redirection via -XX:+DisplayVMOutputToStdout and
    // -XX:+DisplayVMOutputToStderr
    class outputStream : public ResourceObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
       virtual void write(const char* str, size_t len) = 0;
```

### 備考(Notes)
tty 以外に gclog_or_tty という大域変数も用意されている.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    // standard output
                                    // ANSI C++ name collision
    extern outputStream* tty;           // tty output
    extern outputStream* gclog_or_tty;  // stream for gc log if -Xloggc:<f>, or tty
```




### 詳細(Details)
See: [here](../doxygen/classoutputStream.html) for details

---
## <a name="noojgmro7p" id="noojgmro7p">ttyLocker</a>

### 概要(Summary)
defaultStream クラス用のユーティリティ・クラス(StackObjクラス).

defaultStream オブジェクトを使用する際にはスレッド間で排他を取らなければならないが, 
その排他処理(ロックの確保／開放処理)をソースコード上のスコープに合わせて自動で行ってくれる
(See: defaultStream).


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    // advisory locking for the shared tty stream:
    class ttyLocker: StackObj {
```

### 内部構造(Internal structure)
コンストラクタで hold_tty() を呼んでロックを取り, デストラクタで release_tty() を呼んでアンロックするだけ.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
      ttyLocker()  { _holder = hold_tty(); }
      ~ttyLocker() { release_tty(_holder); }
```




### 詳細(Details)
See: [here](../doxygen/classttyLocker.html) for details

---
## <a name="nohYjCc66_" id="nohYjCc66_">ttyUnlocker</a>

### 概要(Summary)
ttyLocker クラス用の補助クラス.

ttyLocker で取得したロックを, ソースコード中のあるスコープの間だけ手放すためのクラス(StackObjクラス).

コメントによると, ロックの取得順序にまつわる問題を回避するために使用される, とのこと.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    // Release the tty lock if it's held and reacquire it if it was
    // locked.  Used to avoid lock ordering problems.
    class ttyUnlocker: StackObj {
```

### 内部構造(Internal structure)
コンストラクタで ttyLocker::release_tty_if_locked() を呼んで (もしロックを取得していれば) 解放する.
デストラクタでは, もしロックを取得していたのであれば hold_tty() を呼んで再取得する.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
      ttyUnlocker()  {
        _was_locked = ttyLocker::release_tty_if_locked();
      }
      ~ttyUnlocker() {
        if (_was_locked) {
          ttyLocker::hold_tty();
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classttyUnlocker.html) for details

---
## <a name="notihfGXvW" id="notihfGXvW">stringStream</a>

### 概要(Summary)
メモリ上のバッファに対して出力するタイプの outputStream
(= sprintf() のようなもの).

なお, バッファ長は必要に応じて自動的に拡張される.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    // for writing to strings; buffer will expand automatically
    class stringStream : public outputStream {
```




### 詳細(Details)
See: [here](../doxygen/classstringStream.html) for details

---
## <a name="noyIkgTkBM" id="noyIkgTkBM">fileStream</a>

### 概要(Summary)
ファイルに書き出すタイプの outputStream.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    class fileStream : public outputStream {
```




### 詳細(Details)
See: [here](../doxygen/classfileStream.html) for details

---
## <a name="noar1m65Qo" id="noar1m65Qo">fdStream</a>

### 概要(Summary)
ファイルに書き出すタイプの outputStream.

fileStream との違いは fdStream はバッファリングしないという点.
コメントによると, このクラスは致命的なエラー時の出力用, とのこと.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    // unlike fileStream, fdStream does unbuffered I/O by calling
    // open() and write() directly. It is async-safe, but output
    // from multiple thread may be mixed together. Used by fatal
    // error handler.
    class fdStream : public outputStream {
```

### 内部構造(Internal structure)
fileStream が FILE* を使用して fopen() と fwrite() で書き出すのに対し,
fdStream は file descriptor を使って open() と write() で書き出す.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.cpp))
    fileStream::fileStream(const char* file_name) {
      _file = fopen(file_name, "w");
```


```
    ((cite: hotspot/src/share/vm/utilities/ostream.cpp))
    void fileStream::write(const char* s, size_t len) {
      if (_file != NULL)  {
        // Make an unused local variable to avoid warning from gcc 4.x compiler.
        size_t count = fwrite(s, 1, len, _file);
```


```
    ((cite: hotspot/src/share/vm/utilities/ostream.cpp))
    fdStream::fdStream(const char* file_name) {
      _fd = open(file_name, O_WRONLY | O_CREAT | O_TRUNC, 0666);
```


```
    ((cite: hotspot/src/share/vm/utilities/ostream.cpp))
    void fdStream::write(const char* s, size_t len) {
      if (_fd != -1) {
        // Make an unused local variable to avoid warning from gcc 4.x compiler.
        size_t count = ::write(_fd, s, (int)len);
```




### 詳細(Details)
See: [here](../doxygen/classfdStream.html) for details

---
## <a name="nokzolv4K2" id="nokzolv4K2">staticBufferStream</a>

### 概要(Summary)
ユーザーから指定されたメモリ上のバッファに書き出すタイプの outputStream.

コメントによると, このクラスは致命的なエラー時の出力用, とのこと.
また, MT safe ではないので出力用のバッファを複数スレッドで共有するとまずい, とのこと.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    // staticBufferStream uses a user-supplied buffer for all formatting.
    // Used for safe formatting during fatal error handling.  Not MT safe.
    // Do not share the stream between multiple threads.
    class staticBufferStream : public outputStream {
```




### 詳細(Details)
See: [here](../doxygen/classstaticBufferStream.html) for details

---
## <a name="norq651IBm" id="norq651IBm">bufferedStream</a>

### 概要(Summary)
メモリ上のバッファに対して出力するタイプの outputStream.

(stringStream との違いは?? 最大バッファ長を明示的に指定できる点?? #TODO)


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    // In the non-fixed buffer case an underlying buffer will be created and
    // managed in C heap. Not MT-safe.
    class bufferedStream : public outputStream {
```




### 詳細(Details)
See: [here](../doxygen/classbufferedStream.html) for details

---
## <a name="noYrBr5Cn3" id="noYrBr5Cn3">networkStream</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

ネットワーク(ソケット)に対して出力するタイプの outputStream.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
    #ifndef PRODUCT
    
    class networkStream : public bufferedStream {
```

### 使われ方(Usage)
IdealGraphPrinter との通信処理に(のみ)使用されている (See: IdealGraphPrinter).

### 備考(Notes)
出力機能だけでなく, ネットワークからデータを読み取る機能も備えていたりする.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.hpp))
        int read(char *buf, size_t len);
```




### 詳細(Details)
See: [here](../doxygen/classnetworkStream.html) for details

---
