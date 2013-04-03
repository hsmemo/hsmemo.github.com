---
layout: default
title: ClassFileParser クラス (ClassFileParser, 及びその補助クラス(ConstantPoolCleaner, NameSigHash, Classfile_LVT_Element, LVT_Hash))
---
[Top](../index.html)

#### ClassFileParser クラス (ClassFileParser, 及びその補助クラス(ConstantPoolCleaner, NameSigHash, Classfile_LVT_Element, LVT_Hash))

これらは, クラスローディング処理用のクラス.
より具体的に言うと, クラスファイルのパース処理を行うクラス (See: [here](no7882m2Z.html) and [here](no7882ALm.html) for details).


### クラス一覧(class list)

  * [ClassFileParser](#no_e6xjKGh)
  * [ConstantPoolCleaner](#noglUjJ9Ru)
  * [NameSigHash](#notiP0eW7A)
  * [Classfile_LVT_Element](#noVJD1Es-a)
  * [LVT_Hash](#noj913T6rW)


---
## <a name="no_e6xjKGh" id="no_e6xjKGh">ClassFileParser</a>

### 概要(Summary)
ClassLoader クラス内で使用される補助クラス.

クラスファイルのパース処理で使用される一時オブジェクト(ValueObjクラス).
実際のクラスファイルのパース処理を行う.

なお, クラスファイル内のデータを読み取る際には ClassFileStream クラスを使用している (See: ClassFileStream).


```
    ((cite: hotspot/src/share/vm/classfile/classFileParser.hpp))
    // Parser for for .class files
    //
    // The bytes describing the class file structure is read from a Stream object
    
    class ClassFileParser VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用箇所(where its instances are used)
ClassLoader::load_classfile() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classClassFileParser.html) for details

---
## <a name="noglUjJ9Ru" id="noglUjJ9Ru">ConstantPoolCleaner</a>

### 概要(Summary)
ClassFileParser クラス内で使用される補助クラス.
クラスファイルのパース処理で使用される一時オブジェクト(StackObjクラス).

パース処理でエラーが起こった場合に, 
パース中にインクリメントした Symbol の参照カウンタ(reference count)を元に戻すためのクラス (See: Symbol).


```
    ((cite: hotspot/src/share/vm/classfile/classFileParser.cpp))
    // This class unreferences constant pool symbols if an error has occurred
    // while parsing the class before it is assigned into the class.
    // If it gets an error after that it is unloaded and the constant pool will
    // be cleaned up then.
    class ConstantPoolCleaner : public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 予め, このクラスのインスタンスに constantPoolHandle を登録しておく.
2. パース中にエラーが起きた場合は set_in_error() を呼んでおく.
3. set_in_error() を呼んだ場合は, デストラクタ処理時に登録した constantPoolHandle の unreference_symbols() を呼び出してくれる.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* ClassFileParser::parseClassFile()
* ClassFileParser::parse_constant_pool()




### 詳細(Details)
See: [here](../doxygen/classConstantPoolCleaner.html) for details

---
## <a name="notiP0eW7A" id="notiP0eW7A">NameSigHash</a>

### 概要(Summary)
ClassFileParser クラス用の補助クラス.
クラスファイルのパース処理で使用される一時オブジェクト(ResourceObjクラス).

パース処理中に使用されるハッシュ. name と signature からなる 2-tuple をキーとする.

クラスファイル中で 「重複したメソッド/フィールド定義がされていないかどうか
(= 名前もシグニチャも等しいメソッド/フィールドが複数定義されていないか)」 を確認するために使われる.

(なお, 正確には NameSigHash はハッシュ内の 1要素を表すクラスであり, 
 NameSigHash** というポインタの配列を作ることでハッシュ(chain hash)を構成する)


```
    ((cite: hotspot/src/share/vm/classfile/classFileParser.cpp))
    class NameSigHash: public ResourceObj {
```

### 内部構造(Internal structure)
以下の関数でハッシュの管理を行っている.

  * initialize_hashtable() 
    
    初期化 (メモリ領域を確保).

  * put_after_lookup() 
    
    ハッシュに指定したエントリを追加 (既に存在した場合は, 何もせずに false を返す)

  * hash() 
    
    ハッシュ値を計算する (put_after_lookup() 内部で使用)


```
    ((cite: hotspot/src/share/vm/classfile/classFileParser.cpp))
    unsigned int hash(Symbol* name, Symbol* sig) {
      unsigned int raw_hash = 0;
      raw_hash += ((unsigned int)(uintptr_t)name) >> (LogHeapWordSize + 2);
      raw_hash += ((unsigned int)(uintptr_t)sig) >> LogHeapWordSize;
    
      return (raw_hash + (unsigned int)(uintptr_t)name) % HASH_ROW_SIZE;
    }
    
    
    void initialize_hashtable(NameSigHash** table) {
      memset((void*)table, 0, sizeof(NameSigHash*) * HASH_ROW_SIZE);
    }
    
    // Return false if the name/sig combination is found in table.
    // Return true if no duplicate is found. And name/sig is added as a new entry in table.
    // The old format checker uses heap sort to find duplicates.
    // NOTE: caller should guarantee that GC doesn't happen during the life cycle
    // of table since we don't expect Symbol*'s to move.
    bool put_after_lookup(Symbol* name, Symbol* sig, NameSigHash** table) {
      assert(name != NULL, "name in constant pool is NULL");
    
      // First lookup for duplicates
      int index = hash(name, sig);
      NameSigHash* entry = table[index];
      while (entry != NULL) {
        if (entry->_name == name && entry->_sig == sig) {
          return false;
        }
        entry = entry->_next;
      }
    
      // No duplicate is found, allocate a new entry and fill it.
      entry = new NameSigHash();
      entry->_name = name;
      entry->_sig = sig;
    
      // Insert into hash table
      entry->_next = table[index];
      table[index] = entry;
    
      return true;
    }
```




### 詳細(Details)
See: [here](../doxygen/classNameSigHash.html) for details

---
## <a name="noVJD1Es-a" id="noVJD1Es-a">Classfile_LVT_Element</a>

### 概要(Summary)
ClassFileParser クラス内で使用される補助クラス.
クラスファイルのパース処理で使用される一時オブジェクト(ValueObjクラス).

クラスファイル中の LocalVariableTable attribute のパース処理を簡単に記述するための補助クラス
(LocalVariableTable attribute を 
 LocalVariableTableElement オブジェクトへとパースする作業内でのみ使用されている (See: ClassFileParser::parse_method())).

このクラスには LocalVariableTable attribute のデータ構造と一致した u2 フィールドが宣言されている.
このため LocalVariableTable attribute table をこのオブジェクトの配列にキャストして操作すると, 
データにフィールド名でアクセスできるようになりコードが分かりやすくなる.


```
    ((cite: hotspot/src/share/vm/classfile/classFileParser.cpp))
    // Class file LocalVariableTable elements.
    class Classfile_LVT_Element VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用例(usage examples)
クラスファイル中の LocalVariableTable attribute を (Classfile_LVT_Element *) にキャストすると, 
配列の index(idx という変数) をインクリメントしていくことで自然と次の要素にアクセスできる.


```
    ((cite: hotspot/src/share/vm/classfile/classFileParser.cpp))
        // To fill LocalVariableTable in
        Classfile_LVT_Element*  cf_lvt;
        LocalVariableTableElement* lvt = m->localvariable_table_start();
    
        for (tbl_no = 0; tbl_no < lvt_cnt; tbl_no++) {
          cf_lvt = (Classfile_LVT_Element *) localvariable_table_start[tbl_no];
          for (idx = 0; idx < localvariable_table_length[tbl_no]; idx++, lvt++) {
            copy_lvt_element(&cf_lvt[idx], lvt);
```

さらに, 実際のコピー処理を行っている copy_lvt_element() 内でも, 
フィールド名でアクセスできるのでコードが分かりやすくなっている.


```
    ((cite: hotspot/src/share/vm/classfile/classFileParser.cpp))
    void copy_lvt_element(Classfile_LVT_Element *src, LocalVariableTableElement *lvt) {
      lvt->start_bci           = Bytes::get_Java_u2((u1*) &src->start_bci);
      lvt->length              = Bytes::get_Java_u2((u1*) &src->length);
      lvt->name_cp_index       = Bytes::get_Java_u2((u1*) &src->name_cp_index);
      lvt->descriptor_cp_index = Bytes::get_Java_u2((u1*) &src->descriptor_cp_index);
      lvt->signature_cp_index  = 0;
      lvt->slot                = Bytes::get_Java_u2((u1*) &src->slot);
    }
```




### 詳細(Details)
See: [here](../doxygen/classClassfile__LVT__Element.html) for details

---
## <a name="noj913T6rW" id="noj913T6rW">LVT_Hash</a>

### 概要(Summary)
ClassFileParser クラス内で使用される補助クラス.
クラスファイル中の LocalVariableTable attribute のパース処理で使用される.

パース処理中に使用されるハッシュ. LocalVariableTableElement オブジェクトをキーとする.

LocalVariableTypeTable attribute の要素に対応する LocalVariableTable attribute の要素を探したり, 
クラスファイル中に重複した LocalVariableTable attribute が定義されていないかどうかを確認するために使われる.

(なお, 正確には LVT_Hash はハッシュ内の 1要素を表すクラスであり, 
 LVT_Hash** というポインタの配列を作ることでハッシュ(chain hash)を構成する)

(なお, このクラスは CHeapObj クラスだが ClassFileParser::parse_method() 内でしか使われないように見えるが... #TODO)


```
    ((cite: hotspot/src/share/vm/classfile/classFileParser.cpp))
    class LVT_Hash: public CHeapObj {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
LVT_put_after_lookup() 内で(のみ)生成されている.

#### 削除箇所(where its instances are deleted)
clear_hashtable() 内で削除されている.

### 内部構造(Internal structure)
以下の関数でハッシュの管理を行っている.

  * initialize_hashtable() 
    
    初期化 (メモリ領域を確保).

  * LVT_put_after_lookup() 
    
    ハッシュに指定したエントリを追加 (既に存在した場合は, 何もせずに false を返す)

  * ... (#TODO)

  * hash() 
    
    ハッシュ値を計算する関数 (LVT_put_after_lookup() 内部等で使用)


```
    ((cite: hotspot/src/share/vm/classfile/classFileParser.cpp))
    unsigned int hash(LocalVariableTableElement *elem) {
      unsigned int raw_hash = elem->start_bci;
    
      raw_hash = elem->length        + raw_hash * 37;
      raw_hash = elem->name_cp_index + raw_hash * 37;
      raw_hash = elem->slot          + raw_hash * 37;
    
      return raw_hash % HASH_ROW_SIZE;
    }
    
    void initialize_hashtable(LVT_Hash** table) {
      for (int i = 0; i < HASH_ROW_SIZE; i++) {
        table[i] = NULL;
      }
    }
    
    void clear_hashtable(LVT_Hash** table) {
      for (int i = 0; i < HASH_ROW_SIZE; i++) {
        LVT_Hash* current = table[i];
        LVT_Hash* next;
        while (current != NULL) {
          next = current->_next;
          current->_next = NULL;
          delete(current);
          current = next;
        }
        table[i] = NULL;
      }
    }
    
    LVT_Hash* LVT_lookup(LocalVariableTableElement *elem, int index, LVT_Hash** table) {
      LVT_Hash* entry = table[index];
    
      /*
       * 3-tuple start_bci/length/slot has to be unique key,
       * so the following comparison seems to be redundant:
       *       && elem->name_cp_index == entry->_elem->name_cp_index
       */
      while (entry != NULL) {
        if (elem->start_bci           == entry->_elem->start_bci
         && elem->length              == entry->_elem->length
         && elem->name_cp_index       == entry->_elem->name_cp_index
         && elem->slot                == entry->_elem->slot
        ) {
          return entry;
        }
        entry = entry->_next;
      }
      return NULL;
    }
    
    // Return false if the local variable is found in table.
    // Return true if no duplicate is found.
    // And local variable is added as a new entry in table.
    bool LVT_put_after_lookup(LocalVariableTableElement *elem, LVT_Hash** table) {
      // First lookup for duplicates
      int index = hash(elem);
      LVT_Hash* entry = LVT_lookup(elem, index, table);
    
      if (entry != NULL) {
          return false;
      }
      // No duplicate is found, allocate a new entry and fill it.
      if ((entry = new LVT_Hash()) == NULL) {
        return false;
      }
      entry->_elem = elem;
    
      // Insert into hash table
      entry->_next = table[index];
      table[index] = entry;
    
      return true;
    }
```




### 詳細(Details)
See: [here](../doxygen/classLVT__Hash.html) for details

---
