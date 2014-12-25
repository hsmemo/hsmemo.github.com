---
layout: default
title: UTF8 クラスおよび UNICODE クラス (UTF8, UNICODE)
---
[Top](../index.html)

#### UTF8 クラスおよび UNICODE クラス (UTF8, UNICODE)



### クラス一覧(class list)

  * [UTF8](#noVZOiIk2G)
  * [UNICODE](#no0Z7M7HV1)


---
## <a name="noVZOiIk2G" id="noVZOiIk2G">UTF8</a>

### 概要(Summary)
UTF8 文字列を扱うためのユーティリティ・クラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/share/vm/utilities/utf8.hpp))
    // Low-level interface for UTF8 strings
    
    class UTF8 : AllStatic {
```

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.

(長さを測ったり, 次の文字の頭出しをしたりする)


```cpp
    ((cite: hotspot/src/share/vm/utilities/utf8.hpp))
      // returns the unicode length of a 0-terminated uft8 string
      static int unicode_length(const char* uft8_str);
    
      // returns the unicode length of a non-0-terminated uft8 string
      static int unicode_length(const char* uft8_str, int len);
    
      // converts a uft8 string to a unicode string
      static void convert_to_unicode(const char* utf8_str, jchar* unicode_buffer, int unicode_length);
    
      // decodes the current utf8 character, stores the result in value,
      // and returns the end of the current uft8 chararacter.
      static char* next(const char* str, jchar* value);
    
      // decodes the current utf8 character, gets the supplementary character instead of
      // the surrogate pair when seeing a supplementary character in string,
      // stores the result in value, and returns the end of the current uft8 chararacter.
      static char* next_character(const char* str, jint* value);
    
      // Utility methods
      static const jbyte* strrchr(const jbyte* base, int length, jbyte c);
      static bool   equal(const jbyte* base1, int length1, const jbyte* base2,int length2);
      static bool   is_supplementary_character(const unsigned char* str);
      static jint   get_supplementary_character(const unsigned char* str);
```




### 詳細(Details)
See: [here](../doxygen/classUTF8.html) for details

---
## <a name="no0Z7M7HV1" id="no0Z7M7HV1">UNICODE</a>

### 概要(Summary)
Unicode (UTF-16) 文字列を扱うためのユーティリティ・クラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).



```cpp
    ((cite: hotspot/src/share/vm/utilities/utf8.hpp))
    // Low-level interface for UNICODE strings
    
    // A unicode string represents a string in the UTF-16 format in which supplementary
    // characters are represented by surrogate pairs. Index values refer to char code
    // units, so a supplementary character uses two positions in a unicode string.
    
    class UNICODE : AllStatic {
```

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.

(なお, 5つとも UTF8 関係のメソッド. 例えば「UTF8 にした場合の長さを調べる」, 「UTF8 に変換する」, 等)


```cpp
    ((cite: hotspot/src/share/vm/utilities/utf8.hpp))
      // returns the utf8 size of a unicode character
      static int utf8_size(jchar c);
    
      // returns the utf8 length of a unicode string
      static int utf8_length(jchar* base, int length);
    
      // converts a unicode string to utf8 string
      static void convert_to_utf8(const jchar* base, int length, char* utf8_buffer);
    
      // converts a unicode string to a utf8 string; result is allocated
      // in resource area unless a buffer is provided.
      static char* as_utf8(jchar* base, int length);
      static char* as_utf8(jchar* base, int length, char* buf, int buflen);
```




### 詳細(Details)
See: [here](../doxygen/classUNICODE.html) for details

---
