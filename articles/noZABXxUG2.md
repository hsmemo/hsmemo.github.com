---
layout: default
title: ResourceArray クラス及び CHeapArray クラス  (ResourceArray, CHeapArray)
---
[Top](../index.html)

#### ResourceArray クラス及び CHeapArray クラス  (ResourceArray, CHeapArray)

これらは, 「配列」や「スタック」といった汎用的なコンテナクラス.

(このクラス群は主に C1 JIT Compiler 向け?? 他で使われているところが見当たらないが... #TODO)

### 概要(Summary)
これらのクラスは「配列」や「スタック」を表す汎用的なコンテナクラス.

なお正確に言うと, ここで定義されているクラスは「配列」を表す以下の 2つだけ.

* ResourceArray クラス
* CHeapArray クラス

ただし, これらを用いて新しいクラスを定義するためのマクロ群が定義されている.

定義されているマクロは以下の通り.

* define_generic_array()
  
  配列クラスを生成するためのマクロ.

  なお, このマクロはこのファイル中の他のマクロを定義するためにしか使われていない (= 他のマクロを定義するための補助マクロ).
  
* define_array()
  
  ResourceArray を継承した配列クラスを生成するためのマクロ.
  
  (主に C1 JIT Compiler 関係のソースコード内で使われている?? #TODO)

* define_stack()
  
  配列クラスを継承して, スタックを表すクラスを生成するためのマクロ.
  
  (主に C1 JIT Compiler 関係のソースコード内で使われている?? #TODO)

* define_resource_list(), define_resource_pointer_list(), define_c_heap_list(), define_c_heap_pointer_list()
  
  ?? (これらについては使用箇所が見当たらない...) #TODO

  ResouceArray や CHeapArray のサブクラスについて, generic_array と stack を同時に定義できるマクロ.
  
### 備考(Notes)
それぞれのマクロの内容は以下の通り.

* define_generic_array()
  
  ベースとなる型と配列クラスの名前, 及び配列要素の型を指定すると, 以下のようなクラス定義を展開してくれる.


```cpp
    ((cite: hotspot/src/share/vm/utilities/array.hpp))
    #define define_generic_array(array_name,element_type, base_class)                        \
      class array_name: public base_class {                                                  \
       protected:                                                                            \
        typedef element_type etype;                                                          \
        enum { esize = sizeof(etype) };                                                      \
                                                                                             \
        void base_remove_at(size_t size, int i) { base_class::remove_at(size, i); }          \
                                                                                             \
       public:                                                                               \
        /* creation */                                                                       \
        array_name() : base_class()                       {}                                 \
        array_name(const int length) : base_class(esize, length) {}                          \
        array_name(const int length, const etype fx)      { initialize(length, fx); }        \
        void initialize(const int length)     { base_class::initialize(esize, length); }     \
        void initialize(const int length, const etype fx) {                                  \
          initialize(length);                                                                \
          for (int i = 0; i < length; i++) ((etype*)_data)[i] = fx;                          \
        }                                                                                    \
                                                                                             \
        /* standard operations */                                                            \
        etype& operator [] (const int i) const {                                             \
          assert(0 <= i && i < length(), "index out of bounds");                             \
          return ((etype*)_data)[i];                                                         \
        }                                                                                    \
                                                                                             \
        int index_of(const etype x) const {                                                  \
          int i = length();                                                                  \
          while (i-- > 0 && ((etype*)_data)[i] != x) ;                                       \
          /* i < 0 || ((etype*)_data)_data[i] == x */                                        \
          return i;                                                                          \
        }                                                                                    \
                                                                                             \
        void sort(int f(etype*, etype*))             { base_class::sort(esize, (ftype)f); }  \
        bool contains(const etype x) const           { return index_of(x) >= 0; }            \
                                                                                             \
        /* deprecated operations - for compatibility with GrowableArray only */              \
        etype  at(const int i) const                 { return (*this)[i]; }                  \
        void   at_put(const int i, const etype x)    { (*this)[i] = x; }                     \
        etype* adr_at(const int i)                   { return &(*this)[i]; }                 \
        int    find(const etype x)                   { return index_of(x); }                 \
      };                                                                                     \
```

* define_array()
  
  配列クラスの名前と配列要素の型を指定すると, 以下のようなクラス定義を展開してくれる.


```cpp
    ((cite: hotspot/src/share/vm/utilities/array.hpp))
    #define define_array(array_name,element_type)                                            \
      define_generic_array(array_name, element_type, ResourceArray)
```

* define_stack()
  
  ベースとなる配列クラスとスタッククラスの名前を指定すると, 以下のようなクラス定義を展開してくれる.


```cpp
    ((cite: hotspot/src/share/vm/utilities/array.hpp))
    #define define_stack(stack_name,array_name)                                              \
      class stack_name: public array_name {                                                  \
       protected:                                                                            \
        int _size;                                                                           \
                                                                                             \
        void grow(const int i, const etype fx) {                                             \
          assert(i >= length(), "index too small");                                          \
          if (i >= size()) expand(esize, i, _size);                                          \
          for (int j = length(); j <= i; j++) ((etype*)_data)[j] = fx;                       \
          _length = i+1;                                                                     \
        }                                                                                    \
                                                                                             \
       public:                                                                               \
        /* creation */                                                                       \
        stack_name() : array_name()                     { _size = 0; }                       \
        stack_name(const int size)                      { initialize(size); }                \
        stack_name(const int size, const etype fx)      { initialize(size, fx); }            \
        void initialize(const int size, const etype fx) {                                    \
          _size = size;                                                                      \
          array_name::initialize(size, fx);                                                  \
          /* _length == size, allocation and size are the same */                            \
        }                                                                                    \
        void initialize(const int size) {                                                    \
          _size = size;                                                                      \
          array_name::initialize(size);                                                      \
          _length = 0;          /* reset length to zero; _size records the allocation */     \
        }                                                                                    \
                                                                                             \
        /* standard operations */                                                            \
        int size() const                             { return _size; }                       \
                                                                                             \
        int push(const etype x) {                                                            \
          int len = length();                                                                \
          if (len >= size()) expand(esize, len, _size);                                      \
          ((etype*)_data)[len] = x;                                                          \
          _length = len+1;                                                                   \
          return len;                                                                        \
        }                                                                                    \
                                                                                             \
        etype pop() {                                                                        \
          assert(!is_empty(), "stack is empty");                                             \
          return ((etype*)_data)[--_length];                                                 \
        }                                                                                    \
                                                                                             \
        etype top() const {                                                                  \
          assert(!is_empty(), "stack is empty");                                             \
          return ((etype*)_data)[length() - 1];                                              \
        }                                                                                    \
                                                                                             \
        void push_all(const stack_name* stack) {                                             \
          const int l = stack->length();                                                     \
          for (int i = 0; i < l; i++) push(((etype*)(stack->_data))[i]);                     \
        }                                                                                    \
                                                                                             \
        etype at_grow(const int i, const etype fx) {                                         \
          if (i >= length()) grow(i, fx);                                                    \
          return ((etype*)_data)[i];                                                         \
        }                                                                                    \
                                                                                             \
        void at_put_grow(const int i, const etype x, const etype fx) {                       \
          if (i >= length()) grow(i, fx);                                                    \
          ((etype*)_data)[i] = x;                                                            \
        }                                                                                    \
                                                                                             \
        void truncate(const int length) {                                                    \
          assert(0 <= length && length <= this->length(), "illegal length");                 \
          _length = length;                                                                  \
        }                                                                                    \
                                                                                             \
        void remove_at(int i)                        { base_remove_at(esize, i); }           \
        void remove(etype x)                         { remove_at(index_of(x)); }             \
                                                                                             \
        /* inserts the given element before the element at index i */                        \
        void insert_before(const int i, const etype el)  {                                   \
          int len = length();                                                                \
          int new_length = len + 1;                                                          \
          if (new_length >= size()) expand(esize, new_length, _size);                        \
          for (int j = len - 1; j >= i; j--) {                                               \
            ((etype*)_data)[j + 1] = ((etype*)_data)[j];                                     \
          }                                                                                  \
          _length = new_length;                                                              \
          at_put(i, el);                                                                     \
        }                                                                                    \
                                                                                             \
        /* inserts contents of the given stack before the element at index i */              \
        void insert_before(const int i, const stack_name *st) {                              \
          if (st->length() == 0) return;                                                     \
          int len = length();                                                                \
          int st_len = st->length();                                                         \
          int new_length = len + st_len;                                                     \
          if (new_length >= size()) expand(esize, new_length, _size);                        \
          int j;                                                                             \
          for (j = len - 1; j >= i; j--) {                                                   \
            ((etype*)_data)[j + st_len] = ((etype*)_data)[j];                                \
          }                                                                                  \
          for (j = 0; j < st_len; j++) {                                                     \
            ((etype*)_data)[i + j] = ((etype*)st->_data)[j];                                 \
          }                                                                                  \
          _length = new_length;                                                              \
        }                                                                                    \
                                                                                             \
        /* deprecated operations - for compatibility with GrowableArray only */              \
        int  capacity() const                        { return size(); }                      \
        void clear()                                 { truncate(0); }                        \
        void trunc_to(const int length)              { truncate(length); }                   \
        int  append(const etype x)                   { return push(x); }                     \
        void appendAll(const stack_name* stack)      { push_all(stack); }                    \
        etype last() const                           { return top(); }                       \
      };                                                                                     \
```

* define_resource_list(), define_resource_pointer_list(), define_c_heap_list(), define_c_heap_pointer_list()
  
  以下のように, generic_array と stack を同時に定義できる.
  

```cpp
    ((cite: hotspot/src/share/vm/utilities/array.hpp))
    #define define_resource_list(element_type)                                               \
      define_generic_array(element_type##Array, element_type, ResourceArray)                 \
      define_stack(element_type##List, element_type##Array)
    
    #define define_resource_pointer_list(element_type)                                       \
      define_generic_array(element_type##Array, element_type *, ResourceArray)               \
      define_stack(element_type##List, element_type##Array)
    
    #define define_c_heap_list(element_type)                                                 \
      define_generic_array(element_type##Array, element_type, CHeapArray)                    \
      define_stack(element_type##List, element_type##Array)
    
    #define define_c_heap_pointer_list(element_type)                                         \
      define_generic_array(element_type##Array, element_type *, CHeapArray)                  \
      define_stack(element_type##List, element_type##Array)
```



### クラス一覧(class list)

  * [ResourceArray](#noEIDSzspf)
  * [CHeapArray](#noDE0-OBzE)


---
## <a name="noEIDSzspf" id="noEIDSzspf">ResourceArray</a>

### 概要(Summary)
いわゆる「配列」を表すユーティリティ・クラス.

このクラスは ResourceObj のサブクラスなので, 一時的な配列が欲しい場合向け.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/utilities/array.hpp))
    class ResourceArray: public ResourceObj {
```



### 詳細(Details)
See: [here](../doxygen/classResourceArray.html) for details

---
## <a name="noDE0-OBzE" id="noDE0-OBzE">CHeapArray</a>

### 概要(Summary)
いわゆる「配列」を表すユーティリティ・クラス.

このクラスは CHeapObj のサブクラスなので, ある程度長期的に利用できる配列が欲しい場合向け.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/utilities/array.hpp))
    class CHeapArray: public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classCHeapArray.html) for details

---
