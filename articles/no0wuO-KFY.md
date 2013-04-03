---
layout: default
title: xmlStream クラス関連のクラス (xmlTextStream, xmlStream)
---
[Top](../index.html)

#### xmlStream クラス関連のクラス (xmlTextStream, xmlStream)



### クラス一覧(class list)

  * [xmlTextStream](#noS7stFQ7J)
  * [xmlStream](#noqW5MJohP)


---
## <a name="noS7stFQ7J" id="noS7stFQ7J">xmlTextStream</a>

### 概要(Summary)
XML 形式のログファイルに対してそのテキスト部分を出力するための outputStream クラス (See: outputStream).

XML のタグに当たる文字を escape する機能を提供している.


```
    ((cite: hotspot/src/share/vm/utilities/xmlstream.hpp))
    // Sub-stream for writing quoted text, as opposed to markup.
    // Characters written to this stream are subject to quoting,
    // as '<' => "&lt;", etc.
    class xmlTextStream : public outputStream {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 xmlStream オブジェクトの _text_init フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(xmlStream クラスの _text_init フィールドは, ポインタ型ではなく実体なので,
 xmlStream オブジェクトの生成時に一緒に生成される)

### 内部構造(Internal structure)
内部の処理的には, xmlStream オブジェクトの単なるラッパーといった感じ.

(内部には, 移譲先として使用する xmlStream オブジェクトを保持している)


```
    ((cite: hotspot/src/share/vm/utilities/xmlstream.hpp))
      xmlStream* _outer_xmlStream;
```

(スーパークラスである outputStream クラスのメソッド以外で) 定義されているメソッドは以下の2つだけ.

どちらも xmlStream オブジェクトの flush() メソッド, write_text() メソッドを呼び出すだけ.


```
    ((cite: hotspot/src/share/vm/utilities/xmlstream.hpp))
       virtual void flush(); // _outer.flush();
       virtual void write(const char* str, size_t len); // _outer->write_text()
```




### 詳細(Details)
See: [here](../doxygen/classxmlTextStream.html) for details

---
## <a name="noqW5MJohP" id="noqW5MJohP">xmlStream</a>

### 概要(Summary)
XML 形式のログファイルに書き出すタイプの outputStream (See: outputStream).


```
    ((cite: hotspot/src/share/vm/utilities/xmlstream.hpp))
    // Output stream for writing XML-structured logs.
    // To write markup, use special calls elem, head/tail, etc.
    // Use the xmlStream::text() stream to write unmarked text.
    // Text written that way will be quoted as necessary using '&lt;', etc.
    // Characters written directly to an xmlStream via print_cr, etc.,
    // are directly written to the encapsulated stream, xmlStream::out().
    // This can be used to produce markup directly, character by character.
    // (Such writes are not checked for markup syntax errors.)
    
    class xmlStream : public outputStream {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 defaultStream オブジェクトの _outer_xmlStream フィールド
* xtty 大域変数 (= tty 大域変数に格納されている defaultStream オブジェクト内の xmlStream オブジェクトのエイリアス)
* 各 IdealGraphPrinter オブジェクトの _xml フィールド

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* defaultStream::init_log() 内
* IdealGraphPrinter::IdealGraphPrinter() 内




### 詳細(Details)
See: [here](../doxygen/classxmlStream.html) for details

---
