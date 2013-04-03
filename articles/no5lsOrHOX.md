---
layout: default
title: OopRecorder およびその補助クラス (OopRecorder, OopRecorder::IndexCache)
---
[Top](../index.html)

#### OopRecorder およびその補助クラス (OopRecorder, OopRecorder::IndexCache)

これらは, JIT コンパイラがコンパイル時に使用した oop 定数情報を記録するためのクラス.

(この情報は, constant table を作ったり, relocation 情報を作ったりする際に使われているもの?? #TODO)


```
    ((cite: hotspot/src/share/vm/code/oopRecorder.hpp))
    // Recording and retrieval of oop relocations in compiled code.
```


### クラス一覧(class list)

  * [OopRecorder](#no5OYAsnKZ)
  * [OopRecorder::IndexCache](#nozFNWES5f)


---
## <a name="no5OYAsnKZ" id="no5OYAsnKZ">OopRecorder</a>

### 概要(Summary)
JIT コンパイル作業中に使用される一時オブジェクト(ResourceObjクラス).

コンパイル中に使用した oop 定数情報を蓄えていくためのクラス.
(oop定数のアドレスを非負の整数と対応付けるマップ(写像), といった感じ.
なお, 0 は常に NULL と対応付けられている.)


```
    ((cite: hotspot/src/share/vm/code/oopRecorder.hpp))
    class OopRecorder : public ResourceObj {
```

### 使われ方(Usage)
OopRecorder オブジェクトは, JIT コンパイルの開始時に作成される.


```
    ((cite: hotspot/src/share/vm/opto/compile.cpp))
    void Compile::Init(int aliaslevel) {
    ...
      // Create Debug Information Recorder to record scopes, oopmaps, etc.
      env()->set_oop_recorder(new OopRecorder(comp_arena()));
```

JIT コンパイル中に, allocate_index() または find_index() で oop 定数情報が蓄えられる (対応する整数値は勝手に割り当てられる).

登録後は, find_index() で oop->int を, handle_at() で int->oop を引ける模様.


```
    ((cite: hotspot/src/share/vm/code/oopRecorder.hpp))
      // Generate a new index on which CodeBlob::oop_addr_at will work.
      // allocate_index and find_index never return the same index,
      // and allocate_index never returns the same index twice.
      // In fact, two successive calls to allocate_index return successive ints.
      int allocate_index(jobject h) {
        return add_handle(h, false);
      }
    
      // For a given jobject, this will return the same index repeatedly.
      // The index can later be given to oop_at to retrieve the oop.
      // However, the oop must not be changed via CodeBlob::oop_addr_at.
      int find_index(jobject h) {
        int index = maybe_find_index(h);
        if (index < 0) {  // previously unallocated
          index = add_handle(h, true);
        }
        return index;
      }
```


```
    ((cite: hotspot/src/share/vm/code/oopRecorder.hpp))
      // Retrieve the oop handle at a given index.
      jobject handle_at(int index);
```

蓄えた情報は, コンパイル完了時に Compile::Fill_buffer() で CodeBuffer にコピーされる.


```
    ((cite: hotspot/src/share/vm/opto/output.cpp))
    void Compile::Fill_buffer() {
    ...
      cb->initialize_oop_recorder(env()->oop_recorder());
```

なお, この情報は最終的には OopRecorder::copy_to() で CodeBuffer から nmethod にコピーされている.


```
    ((cite: hotspot/src/share/vm/asm/codeBuffer.hpp))
      void copy_oops_to(nmethod* nm) {
        if (!oop_recorder()->is_unused()) {
          oop_recorder()->copy_to(nm);
        }
```




### 詳細(Details)
See: [here](../doxygen/classOopRecorder.html) for details

---
## <a name="nozFNWES5f" id="nozFNWES5f">OopRecorder::IndexCache</a>

### 概要(Summary)
OopRecorder クラス内で使用される補助クラス.

OopRecorder::find_index() を高速化するためのキャッシュ.


```
    ((cite: hotspot/src/share/vm/code/oopRecorder.hpp))
      // leaky hash table of handle => index, to help detect duplicate insertion
      class IndexCache: public ResourceObj {
        // This class is only used by the OopRecorder class.
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
OopRecorder オブジェクトの _indexes フィールドに格納されており,
jobject から index への(やや不正確な)対応を記録している.


```
    ((cite: hotspot/src/share/vm/code/oopRecorder.hpp))
      IndexCache*               _indexes;  // map: jobject -> its probable index
```




### 詳細(Details)
See: [here](../doxygen/classOopRecorder_1_1IndexCache.html) for details

---
