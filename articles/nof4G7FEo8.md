---
layout: default
title: constMethodOop クラス関連のクラス (CheckedExceptionElement, LocalVariableTableElement, constMethodOopDesc)
---
[Top](../index.html)

#### constMethodOop クラス関連のクラス (CheckedExceptionElement, LocalVariableTableElement, constMethodOopDesc)

これらは, メソッドに関する情報を管理するクラス.


### クラス一覧(class list)

  * [constMethodOopDesc](#nomm7U0gEJ)
  * [CheckedExceptionElement](#no9PszKXLP)
  * [LocalVariableTableElement](#nocrwV3Qvn)


---
## <a name="nomm7U0gEJ" id="nomm7U0gEJ">constMethodOopDesc</a>

### 概要(Summary)
methodOopDesc クラス用の補助クラス.

メソッドに関する情報のうち read-only なものを管理するためのクラス.
具体的には以下のようなデータを管理している (methodOopDesc の構造も参照).

  * 実際のバイトコード(メソッドのbody部)
  * line number table(line_number_table)情報        (なお, このデータ量は多くなりがちなので圧縮して保持している)
  * checked exceptions table 情報                   (なお, このデータはたいてい 2 個以下なので特に圧縮していない模様)
  * local variable table(local_variable_table)情報  (なお, このデータはたいてい空なので特に圧縮していないらしい模様)

(なお, バイトコード以外の情報についてはオプショナルなので存在しない可能性がある.
存在するかどうかは has_linenumber_table() 等のメソッドで確認できる)


```cpp
    ((cite: hotspot/src/share/vm/oops/constMethodOop.hpp))
    class constMethodOopDesc : public oopDesc {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
methodOopDesc オブジェクトの _constMethod フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.hpp))
    class methodOopDesc : public oopDesc {
    ...
      constMethodOop    _constMethod;                // Method read-only data.
```

(なお, 逆に constMethodOop 側から対応する methodOop を参照するには _method フィールドを用いればいい模様)


```cpp
    ((cite: hotspot/src/share/vm/oops/constMethodOop.hpp))
    class constMethodOopDesc : public oopDesc {
    ...
      // Backpointer to non-const methodOop (needed for some JVMTI operations)
      methodOop         _method;
```

#### 生成箇所(where its instances are created)
oopFactory::new_constMethod() というファクトリメソッドが用意されており, その中で(のみ)生成されている.

### 内部構造(Internal structure)
内部のレイアウトは以下に示す通り

(各1列が1word相当. 1行に複数書かれている所は u2 のものが詰め込まれていたりする箇所)
(<= u2 だから 64bit だとレイアウト変わるのでは？).

なおコメントによると, 
メソッドは本当に大量に存在しているので constMethodOopDesc をコンパクトにまとめるのはメモリ使用量を下げる上でかなり重要, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/oops/constMethodOop.hpp))
    // An constMethodOop represents portions of a Java method which
    // do not vary.
    //
    // Memory layout (each line represents a word). Note that most
    // applications load thousands of methods, so keeping the size of this
    // structure small has a big impact on footprint.
    //
    // |------------------------------------------------------|
    // | header                                               |
    // | klass                                                |
    // |------------------------------------------------------|
    // | fingerprint 1                                        |
    // | fingerprint 2                                        |
    // | method                         (oop)                 |
    // | stackmap_data                  (oop)                 |
    // | exception_table                (oop)                 |
    // | constMethod_size                                     |
    // | interp_kind  | flags    | code_size                  |
    // | name index              | signature index            |
    // | method_idnum            | generic_signature_index    |
    // |------------------------------------------------------|
    // |                                                      |
    // | byte codes                                           |
    // |                                                      |
    // |------------------------------------------------------|
    // | compressed linenumber table                          |
    // |  (see class CompressedLineNumberReadStream)          |
    // |  (note that length is unknown until decompressed)    |
    // |  (access flags bit tells whether table is present)   |
    // |  (indexed from start of constMethodOop)              |
    // |  (elements not necessarily sorted!)                  |
    // |------------------------------------------------------|
    // | localvariable table elements + length (length last)  |
    // |  (length is u2, elements are 6-tuples of u2)         |
    // |  (see class LocalVariableTableElement)               |
    // |  (access flags bit tells whether table is present)   |
    // |  (indexed from end of contMethodOop)                 |
    // |------------------------------------------------------|
    // | checked exceptions elements + length (length last)   |
    // |  (length is u2, elements are u2)                     |
    // |  (see class CheckedExceptionElement)                 |
    // |  (access flags bit tells whether table is present)   |
    // |  (indexed from end of constMethodOop)                |
    // |------------------------------------------------------|
```

### 備考(Notes)
なお, 実際の使用箇所では constMethodOop という別名(もしくはラッパークラス)で使われることが多い (See: constMethodOop).




### 詳細(Details)
See: [here](../doxygen/classconstMethodOopDesc.html) for details

---
## <a name="no9PszKXLP" id="no9PszKXLP">CheckedExceptionElement</a>

### 概要(Summary)
methodOopDesc 内 (正確にはそこから参照されている constMethodOop 内) で使用される補助クラス.

そのメソッドが投げうる検査例外(checked exception)情報を格納している
(言い換えると, クラスファイルにおける method_info 中の Exceptions Attribute の情報を格納しておくクラス).


```cpp
    ((cite: hotspot/src/share/vm/oops/constMethodOop.hpp))
    // Utitily class decribing elements in checked exceptions table inlined in methodOop.
    class CheckedExceptionElement VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ClassFileParser::parse_method() 内で methodOop を作成するときに一緒に作成される (See: [here](no2114rPX.html) for details).

(なお, CheckedExceptionElement オブジェクトに格納される情報は ClassFileParser::parse_checked_exceptions() でパースされている)

#### 参考(for your information): ClassFileParser::parse_method()
See: [here](no17119ley.html) for details
### 内部構造(Internal structure)
単なる構造体のようなクラス.
内部には以下の public フィールドのみを持つ (そしてメソッドはない).


```cpp
    ((cite: hotspot/src/share/vm/oops/constMethodOop.hpp))
      u2 class_cp_index;
```




### 詳細(Details)
See: [here](../doxygen/classCheckedExceptionElement.html) for details

---
## <a name="nocrwV3Qvn" id="nocrwV3Qvn">LocalVariableTableElement</a>

### 概要(Summary)
methodOopDesc 内 (正確にはそこから参照されている constMethodOop 内) で使用される補助クラス.

そのメソッドの局所変数の情報を格納している
(言い換えると, クラスファイルにおける method_info 中の LocalVariableTable Attribute の情報を格納しておくクラス).


```cpp
    ((cite: hotspot/src/share/vm/oops/constMethodOop.hpp))
    // Utitily class decribing elements in local variable table inlined in methodOop.
    class LocalVariableTableElement VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ClassFileParser::parse_method() 内で methodOop を作成するときに一緒に作成されている (See: [here](no2114rPX.html) for details).

#### 参考(for your information): ClassFileParser::parse_method()
See: [here](no17119ley.html) for details
### 内部構造(Internal structure)
単なる構造体のようなクラス.
内部には以下の public フィールドのみを持つ (そしてメソッドはない).


```cpp
    ((cite: hotspot/src/share/vm/oops/constMethodOop.hpp))
      u2 start_bci;
      u2 length;
      u2 name_cp_index;
      u2 descriptor_cp_index;
      u2 signature_cp_index;
      u2 slot;
```




### 詳細(Details)
See: [here](../doxygen/classLocalVariableTableElement.html) for details

---
