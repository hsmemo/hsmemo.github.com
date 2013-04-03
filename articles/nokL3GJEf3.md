---
layout: default
title: Class Data Sharing (CDS) の統計情報出力のための補助クラス (ClassifyObjectClosure, ClassifyInstanceKlassClosure, ClearAllocCountClosure)
---
[Top](../index.html)

#### Class Data Sharing (CDS) の統計情報出力のための補助クラス (ClassifyObjectClosure, ClassifyInstanceKlassClosure, ClearAllocCountClosure)

これらは, トラブルシューティング用のクラス.

Class Data Sharing (CDS) 機能に関する統計情報の出力処理で使用される (See: [here](no2114Sn1.html) for details).

なお, これらのは全てクラスデータのダンプ処理を行う VM_PopulateDumpSharedSpace::doit() 内で使用される
(より正確に言うと, この関数の最後で呼び出される print_contents() 内で使用される).

また, これらは product オプションである PrintSharedSpaces オプションが指定されている場合にのみ使用される.



### クラス一覧(class list)

  * [ClassifyObjectClosure](#no1bXvsfl0)
  * [ClassifyInstanceKlassClosure](#noElKSuTf7)
  * [ClearAllocCountClosure](#noEEFyIbHJ)


---
## <a name="no1bXvsfl0" id="no1bXvsfl0">ClassifyObjectClosure</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する product オプションが指定されている場合にのみ使用される) (See: PrintSharedSpaces).

Class Data Sharing (CDS) 機能に関する統計情報出力用の補助クラス (Closure クラス).

ヒープ領域内を辿って, どういった種別のオブジェクトが何個／何バイト存在しているかを調べる.

(ついでに, 各クラス毎に, そのクラスのインスタンスがどれだけ存在しているかという情報も記録している.
 この情報は ClassifyInstanceKlassClosure が使用する.
 (See: ClassifyInstanceKlassClosure))


```
    ((cite: hotspot/src/share/vm/memory/classify.hpp))
    // Classify objects by type and keep counts.
    // Print the count and space taken for each type.
    
    
    class ClassifyObjectClosure : public ObjectClosure {
```

### 使われ方(Usage)
print_contents() 内で(のみ)使用されている.

### 備考(Notes)
分類分けに使用する種別は以下の通り.


```
    ((cite: hotspot/src/share/vm/memory/classify.hpp))
    typedef enum oop_type {
      unknown_type,
      instance_type,
      instanceRef_type,
      objArray_type,
      symbol_type,
      klass_type,
      instanceKlass_type,
      method_type,
      constMethod_type,
      methodData_type,
      constantPool_type,
      constantPoolCache_type,
      typeArray_type,
      compiledICHolder_type,
      number_object_types
    } object_type;
```




### 詳細(Details)
See: [here](../doxygen/classClassifyObjectClosure.html) for details

---
## <a name="noElKSuTf7" id="noElKSuTf7">ClassifyInstanceKlassClosure</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する product オプションが指定されている場合にのみ使用される) (See: PrintSharedSpaces).

Class Data Sharing (CDS) 機能に関する統計情報出力用の補助クラス (Closure クラス).

Perm 領域内を辿って, それぞれのクラス毎にインスタンスがどれだけあるかを出力する.


```
    ((cite: hotspot/src/share/vm/memory/classify.hpp))
    // Count objects using the alloc_count field in the object's klass
    // object.
    
    class ClassifyInstanceKlassClosure : public ClassifyObjectClosure {
```

### 使われ方(Usage)
print_contents() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classClassifyInstanceKlassClosure.html) for details

---
## <a name="noEEFyIbHJ" id="noEEFyIbHJ">ClearAllocCountClosure</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する product オプションが指定されている場合にのみ使用される) (See: PrintSharedSpaces).

Class Data Sharing (CDS) 機能に関する統計情報出力用の補助クラス (Closure クラス).

ClassifyObjectClosure が記録したクラス毎のインスタンス量の情報を消去する.


```
    ((cite: hotspot/src/share/vm/memory/classify.hpp))
    // Clear the alloc_count fields in all classes so that the count can be
    // restarted.
    
    class ClearAllocCountClosure : public ObjectClosure {
```

### 使われ方(Usage)
print_contents() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classClearAllocCountClosure.html) for details

---
