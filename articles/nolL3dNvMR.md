---
layout: default
title: TemplateInterpreter クラス関連のクラス (EntryPoint, DispatchTable, TemplateInterpreter)
---
[Top](../index.html)

#### TemplateInterpreter クラス関連のクラス (EntryPoint, DispatchTable, TemplateInterpreter)

これらは, Template Interpreter 用のクラス (#ifndef CC_INTERP 時にしか定義されない).

より具体的に言うと, 実際の Template Interpreter として働く Interpreter クラス. (See: [here](no7882AgC.html) for details)


```
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.hpp))
    // This file contains the platform-independent parts
    // of the template interpreter and the template interpreter generator.
    
    #ifndef CC_INTERP
```


### クラス一覧(class list)

  * [TemplateInterpreter](#nobzKgCmLh)
  * [EntryPoint](#nou917WX3f)
  * [DispatchTable](#nomP-r53eU)


---
## <a name="nobzKgCmLh" id="nobzKgCmLh">TemplateInterpreter</a>

### 概要(Summary)
Template Interpreter 用のクラス (#ifndef CC_INTERP 時にしか定義されない).

Template Interpreter において Interpreter の初期化や生成されたコードの管理を行うためのクラス
(= Template Interpreter 用の Interpreter クラス). (See: [here](no7882AgC.html) for details)


```
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.hpp))
    class TemplateInterpreter: public AbstractInterpreter {
```




### 詳細(Details)
See: [here](../doxygen/classTemplateInterpreter.html) for details

---
## <a name="nou917WX3f" id="nou917WX3f">EntryPoint</a>

### 概要(Summary)
TemplateInterpreter クラス内で使用される補助クラス.

TemplateInterpreter 用のコードレットのエントリポイントの管理を担当する
(Template Interpreter のコードレットは TOS に応じたエントリーポイントを持つ.
これら number_of_states 個のエントリーポイントを管理するためのクラス).


```
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.hpp))
    //------------------------------------------------------------------------------------------------------------------------
    // A little wrapper class to group tosca-specific entry points into a unit.
    // (tosca = Top-Of-Stack CAche)
    
    class EntryPoint VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
内部的には本当に number_of_states 個分の address 配列を持っているだけ.


```
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.hpp))
      address _entry[number_of_states];
```




### 詳細(Details)
See: [here](../doxygen/classEntryPoint.html) for details

---
## <a name="nomP-r53eU" id="nomP-r53eU">DispatchTable</a>

### 概要(Summary)
TemplateInterpreter クラス内で使用される補助クラス.

Template Interpreter の dispatch table を管理するためのクラス.

(なお dispatch table とはメモリアドレスの二次元配列.
 各バイトコードと TosState を index とし, コードレットのエントリポイントを格納している.
 各バイトコードを各 TOS 状態で実行する際にどこにジャンプすればいいか, という情報を表す)


```
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.hpp))
    //------------------------------------------------------------------------------------------------------------------------
    // A little wrapper class to group tosca-specific dispatch tables into a unit.
    
    class DispatchTable VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている (See: [here](no7882rhh.html) for details).

* TemplateInterpreter クラスの _active_table フィールド (static フィールド)
* TemplateInterpreter クラスの _normal_table フィールド (static フィールド)
* TemplateInterpreter クラスの _safept_table フィールド (static フィールド)


```
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.hpp))
      static DispatchTable _active_table;                           // the active    dispatch table (used by the interpreter for dispatch)
      static DispatchTable _normal_table;                           // the normal    dispatch table (used to set the active table in normal mode)
      static DispatchTable _safept_table;                           // the safepoint dispatch table (used to set the active table for safepoints)
```




### 詳細(Details)
See: [here](../doxygen/classDispatchTable.html) for details

---
