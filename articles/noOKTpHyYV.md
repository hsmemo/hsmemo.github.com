---
layout: default
title: デバッグ用の関数/マクロのための補助クラス (FormatBuffer, Command, LookForRefInGenClosure, LookForRefInObjectClosure, FindClassObjectClosure)
---
[Top](../index.html)

#### デバッグ用の関数/マクロのための補助クラス (FormatBuffer, Command, LookForRefInGenClosure, LookForRefInObjectClosure, FindClassObjectClosure)

これらは, デバッグ用(開発時用)の機能の中で使用されるユーティリティ・クラス.


### クラス一覧(class list)

  * [FormatBuffer](#noHpnnh-Qo)
  * [Command](#noCyMsVrdr)
  * [LookForRefInGenClosure](#noZ-HaUs7d)
  * [LookForRefInObjectClosure](#noSKTjt32k)
  * [FindClassObjectClosure](#noodDOvFDe)


---
## <a name="noHpnnh-Qo" id="noHpnnh-Qo">FormatBuffer</a>

### 概要(Summary)
sprintf() のような働きをするユーティリティ・クラス.

コンストラクタで渡された書式付き文字列に従って文字列を生成する.


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.hpp))
    // Simple class to format the ctor arguments into a fixed-sized buffer.
    template <size_t bufsz = 256>
    class FormatBuffer {
```

なお err_msg や hrs_err_msg という型も使われるが, これは FormatBuffer の別名.


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.hpp))
    // Used to format messages for assert(), guarantee(), fatal(), etc.
    typedef FormatBuffer<> err_msg;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
    // Large buffer for some cases where the output might be larger than normal.
    #define HRS_ERR_MSG_BUFSZ 512
    typedef FormatBuffer<HRS_ERR_MSG_BUFSZ> hrs_err_msg;
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コンストラクタ引数として書式付き文字列を渡す
(なおテンプレート引数はバッファ長. デフォルトは 256).

生成された文字列は FormatBuffer オブジェクトを char* にキャストすることで取得できる.


```cpp
    ((cite: hotspot/src/share/vm/runtime/frame.cpp))
        values.describe(-1, MAX2(sp(), fp()),
                        FormatBuffer<1024>("#%d method %s @ %d", frame_no, m->name_and_sig_as_C_string(), bci), 2);
```

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

(特に assert() や guarantee() 等に渡すエラーメッセージの生成処理が多い).

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を保持する.


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.hpp))
      char _buf[bufsz];
```

このフィールドは, char* にキャストすることで取得できる.


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.hpp))
      operator const char *() const { return _buf; }
```




### 詳細(Details)
See: [here](../doxygen/classFormatBuffer.html) for details

---
## <a name="noCyMsVrdr" id="noCyMsVrdr">Command</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

hotspot/src/share/vm/utilities/debug.cpp 内で定義されているデバッグ用の関数群(blob(), dump_vtable(), nm(), etc)内で使用される補助クラス.

これらの関数の前準備と後始末を行う.


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.cpp))
    #ifndef PRODUCT
    // All debug entries should be wrapped with a stack allocated
    // Command object. It makes sure a resource mark is set and
    // flushes the logfile to prevent file sharing problems.
    
    class Command : public StackObj {
```

### 使われ方(Usage)
コード中で Command 型の局所変数を宣言するだけ.


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.cpp))
    extern "C" void universe() {
      Command c("universe");
      Universe::print();
```

### 内部構造(Internal structure)
コンストラクタで Debugging オプションの値を true に変更し, デストラクタで元に戻している.

(ついでに, デバッグ用の関数が呼び出されたことをコンストラクタ内で tty に出力し, デストラクタ内で tty をフラッシュしている)

(なお, Debugging は現在デバッグ中であることを示すオプション.
 true にすると VMError::report_and_die() 等が死なずにリターンするようになる)


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.cpp))
      Command(const char* str) {
        debug_save = Debugging;
        Debugging = true;
        if (level++ > 0)  return;
        tty->cr();
        tty->print_cr("\"Executing %s\"", str);
      }
    
      ~Command() { tty->flush(); Debugging = debug_save; level--; }
```




### 詳細(Details)
See: [here](../doxygen/classCommand.html) for details

---
## <a name="noZ-HaUs7d" id="noZ-HaUs7d">LookForRefInGenClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

デバッグ用の関数である findref() 内で(のみ)使用される補助クラス.

Java ヒープ中を探索し, 指定された oop(ポインタ値) と一致するものを見つけたら tty に出力する Closure.


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.cpp))
    #ifndef PRODUCT
    ...
    class LookForRefInGenClosure : public OopsInGenClosure {
```




### 詳細(Details)
See: [here](../doxygen/classLookForRefInGenClosure.html) for details

---
## <a name="noSKTjt32k" id="noSKTjt32k">LookForRefInObjectClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

デバッグ用の関数である findref() 内で(のみ)使用される補助クラス.

指定されたオブジェクト中の全ての oop に対して LookForRefInGenClosure を呼び出す Closure.


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.cpp))
    #ifndef PRODUCT
    ...
    class LookForRefInObjectClosure : public ObjectClosure {
```




### 詳細(Details)
See: [here](../doxygen/classLookForRefInObjectClosure.html) for details

---
## <a name="noodDOvFDe" id="noodDOvFDe">FindClassObjectClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

デバッグ用の関数である findclass() 内で(のみ)使用される補助クラス.

perm 領域中を探索し, 指定された名前(文字列)と一致するクラス名を持つ Klass オブジェクトを tty に出力する Closure.


```cpp
    ((cite: hotspot/src/share/vm/utilities/debug.cpp))
    #ifndef PRODUCT
    ...
    class FindClassObjectClosure: public ObjectClosure {
```





### 詳細(Details)
See: [here](../doxygen/classFindClassObjectClosure.html) for details

---
