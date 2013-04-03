---
layout: default
title: TemplateTable クラス関連のクラス (Template, TemplateTable)
---
[Top](../index.html)

#### TemplateTable クラス関連のクラス (Template, TemplateTable)

これらは, Template Interpreter 用のクラス (#ifndef CC_INTERP 時にしか定義されない).

Template Interpreter が使用するコードレットの生成を担当する.


```
    ((cite: hotspot/src/share/vm/interpreter/templateTable.hpp))
    #ifndef CC_INTERP
    // All the necessary definitions used for (bytecode) template generation. Instead of
    // spreading the implementation functionality for each bytecode in the interpreter
    // and the snippet generator, a template is assigned to each bytecode which can be
    // used to generate the bytecode's implementation if needed.
```


### クラス一覧(class list)

  * [TemplateTable](#noxFepDyB0)
  * [Template](#noFgafabI5)


---
## <a name="noxFepDyB0" id="noxFepDyB0">TemplateTable</a>

### 概要(Summary)
Template Interpreter 用のクラス (#ifndef CC_INTERP 時にしか定義されない).
このクラスが, Template Interpreter における実際の各bytecodeでの挙動を定めている
(C++ Interpreter における BytecodeInterpreter クラスに相当). (See: [here](no7882AgC.html), [here](no3059SwU.html) and [here](no7882bnt.html) for details)

より具体的に言うと, 
Template Interpreter 用のコードレットの生成処理を行うクラス (より正確には, そのための機能を納めた名前空間(AllStatic クラス))


```
    ((cite: hotspot/src/share/vm/interpreter/templateTable.hpp))
    // The TemplateTable defines all Templates and provides accessor functions
    // to get the template for a given bytecode.
    
    class TemplateTable: AllStatic {
```

### 内部構造(Internal structure)
各バイトコード用のマシン語列を生成するメソッドが定義されている
(e.g. TemplateTable::aload(), TemplateTable::invokevirtual(), TemplateTable::getfield(), etc).

(なお, これらのメソッドの中身は cpu/ ディレクトリ下で実装されている)




### 詳細(Details)
See: [here](../doxygen/classTemplateTable.html) for details

---
## <a name="noFgafabI5" id="noFgafabI5">Template</a>

### 概要(Summary)
Template Interpreter 用のクラス (#ifndef CC_INTERP 時にしか定義されない).

TemplateTable クラス内で使用される補助クラス. 
各コードレットの生成方法についての情報を格納している.
1つの Template オブジェクトが 1つのコードレットに担当する.


```
    ((cite: hotspot/src/share/vm/interpreter/templateTable.hpp))
    // A Template describes the properties of a code template for a given bytecode
    // and provides a generator to generate the code template.
    
    class Template VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* Template クラスの _template_table フィールド (static フィールド)
  
  (正確には, このフィールドは Template の配列を格納するフィールド.
  この中に, wide prefix 無しのバイトコードに対応する全ての Template オブジェクトが格納されている.
  配列長は Bytecodes::number_of_codes)

* Template クラスの _template_table_wide フィールド (static フィールド)
  
  (正確には, このフィールドは Template の配列を格納するフィールド.
  この中に, wide prefix 有りのバイトコードに対応する全ての Template オブジェクトが格納されている.
  配列長は Bytecodes::number_of_codes)

#### 生成箇所(where its instances are created)
(上記の配列フィールドは全て, ポインタ型ではなく実体なので, 配列用のメモリ領域は初期段階で自動的に確保される)

そのメモリ領域中に個別の Template オブジェクトを書き込む作業は Template::initialize() 内で(のみ)行われている.

### 内部構造(Internal structure)
内部的には, 以下のフィールドを保持している.

これらにコードレットの情報(コードレットに入るときの TOS 状態/出るときの TOS 状態, 
対応するマシン語列を生成する生成関数の情報, 等)を格納している.


```
    ((cite: hotspot/src/share/vm/interpreter/templateTable.hpp))
      int       _flags;                              // describes interpreter template properties (bcp unknown)
      TosState  _tos_in;                             // tos cache state before template execution
      TosState  _tos_out;                            // tos cache state after  template execution
      generator _gen;                                // template code generator
      int       _arg;                                // argument for template code generator
```

(なお, Template クラス自体はコード生成は行わない.
Template クラスはコードを生成するための関数等の情報を記録しているだけで, 
実際の生成処理はここに記録されている関数 (= TemplateTable のメソッド) が行う.
(See: Template::generate()))




### 詳細(Details)
See: [here](../doxygen/classTemplate.html) for details

---
