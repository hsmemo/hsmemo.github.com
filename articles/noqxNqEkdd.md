---
layout: default
title: ciMethodData クラス関連のクラス (ciBitData, ciCounterData, ciJumpData, ciReceiverTypeData, ciVirtualCallData, ciRetData, ciBranchData, ciArrayData, ciMultiBranchData, ciArgInfoData, ciMethodData)
---
[Top](../index.html)

#### ciMethodData クラス関連のクラス (ciBitData, ciCounterData, ciJumpData, ciReceiverTypeData, ciVirtualCallData, ciRetData, ciBranchData, ciArrayData, ciMultiBranchData, ciArgInfoData, ciMethodData)

これらは, methodDataOopDesc 用の ciObject クラス.

なお ciProfileData という型も使われるが, これは ProfileData の別名.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    typedef ProfileData ciProfileData;
```


### クラス一覧(class list)

  * [ciBitData](#noR8sL929Z)
  * [ciCounterData](#novJ90SFWL)
  * [ciJumpData](#noXdMgbZFa)
  * [ciReceiverTypeData](#noYPWgnscp)
  * [ciVirtualCallData](#noAaYoudsd)
  * [ciRetData](#now2BmILPf)
  * [ciBranchData](#noyXjuYATq)
  * [ciArrayData](#noBsAxNdif)
  * [ciMultiBranchData](#noN6Ey380a)
  * [ciArgInfoData](#no_fq7wPTH)
  * [ciMethodData](#noArHbXFt0)


---
## <a name="noR8sL929Z" id="noR8sL929Z">ciBitData</a>

### 概要(Summary)
ciMethodData クラス用の補助クラス.

ciMethodData クラスから BitData オブジェクト内のプロファイル情報を扱うためのラッパークラス.
1つの ciBitData オブジェクトが 1つの BitData オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciBitData : public BitData {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* ciMethodData::data_at()
* ciMethodData::bci_to_data()

### 内部構造(Internal structure)
何も定義されていない (なので中身はスーパークラスである BitData と全く同じ).




### 詳細(Details)
See: [here](../doxygen/classciBitData.html) for details

---
## <a name="novJ90SFWL" id="novJ90SFWL">ciCounterData</a>

### 概要(Summary)
ciMethodData クラス用の補助クラス.

ciMethodData クラスから CounterData オブジェクト内のプロファイル情報を扱うためのラッパークラス.
1つの ciCounterData オブジェクトが 1つの CounterData オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciCounterData : public CounterData {
```

### 使われ方(Usage)
ciMethodData::data_at() 内で(のみ)生成されている.

### 内部構造(Internal structure)
何も定義されていない (なので中身はスーパークラスである CounterData と全く同じ).




### 詳細(Details)
See: [here](../doxygen/classciCounterData.html) for details

---
## <a name="noXdMgbZFa" id="noXdMgbZFa">ciJumpData</a>

### 概要(Summary)
ciMethodData クラス用の補助クラス.

ciMethodData クラスから JumpData オブジェクト内のプロファイル情報を扱うためのラッパークラス.
1つの ciJumpData オブジェクトが 1つの JumpData オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciJumpData : public JumpData {
```

### 使われ方(Usage)
ciMethodData::data_at() 内で(のみ)生成されている.

### 内部構造(Internal structure)
何も定義されていない (なので中身はスーパークラスである JumpData と全く同じ).




### 詳細(Details)
See: [here](../doxygen/classciJumpData.html) for details

---
## <a name="noYPWgnscp" id="noYPWgnscp">ciReceiverTypeData</a>

### 概要(Summary)
ciMethodData クラス用の補助クラス.

ciMethodData クラスから ReceiverTypeData オブジェクト内のプロファイル情報を扱うためのラッパークラス.
1つの ciReceiverTypeData オブジェクトが 1つの ReceiverTypeData オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciReceiverTypeData : public ReceiverTypeData {
```

### 使われ方(Usage)
ciMethodData::data_at() 内で(のみ)生成されている.

### 内部構造(Internal structure)
このクラスは内部にポインタ情報を含むため, 
ProfileData::translate_from() をオーバーライドしている
(See: ciReceiverTypeData::translate_from()).

使用時には, ciReceiverTypeData::receiver() で型情報を (ciKlass オブジェクトとして) 取得できる.
また, ciReceiverTypeData::set_receiver() で型情報を変更することもできる.




### 詳細(Details)
See: [here](../doxygen/classciReceiverTypeData.html) for details

---
## <a name="noAaYoudsd" id="noAaYoudsd">ciVirtualCallData</a>

### 概要(Summary)
ciMethodData クラス用の補助クラス.

ciMethodData クラスから VirtualCallData オブジェクト内のプロファイル情報を扱うためのラッパークラス.
1つの ciVirtualCallData オブジェクトが 1つの VirtualCallData オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciVirtualCallData : public VirtualCallData {
```

### 使われ方(Usage)
ciMethodData::data_at() 内で(のみ)生成されている.

### 内部構造(Internal structure)
実際の処理は ciReceiverTypeData に丸投げしている

(スーパークラスである VirtualCallData と同様,  現状では ciReceiverTypeData クラスと全く同じ.
 ただし, VirtualCallData と違って ciReceiverTypeData がスーパークラスではないので, 
 無理矢理 ciReceiverTypeData にキャストして呼び出している).


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
      // Fake multiple inheritance...  It's a ciReceiverTypeData also.
      ciReceiverTypeData* rtd_super() { return (ciReceiverTypeData*) this; }
```


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
      void set_receiver(uint row, ciKlass* recv) {
        rtd_super()->set_receiver(row, recv);
      }
    
      ciKlass* receiver(uint row) {
        return rtd_super()->receiver(row);
      }
    
      // Copy & translate from oop based VirtualCallData
      virtual void translate_from(ProfileData* data) {
        rtd_super()->translate_receiver_data_from(data);
      }
```




### 詳細(Details)
See: [here](../doxygen/classciVirtualCallData.html) for details

---
## <a name="now2BmILPf" id="now2BmILPf">ciRetData</a>

### 概要(Summary)
ciMethodData クラス用の補助クラス.

ciMethodData クラスから RetData オブジェクト内のプロファイル情報を扱うためのラッパークラス.
1つの ciRetData オブジェクトが 1つの RetData オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciRetData : public RetData {
```

### 使われ方(Usage)
ciMethodData::data_at() 内で(のみ)生成されている.

### 内部構造(Internal structure)
何も定義されていない (なので中身はスーパークラスである RetData と全く同じ).




### 詳細(Details)
See: [here](../doxygen/classciRetData.html) for details

---
## <a name="noyXjuYATq" id="noyXjuYATq">ciBranchData</a>

### 概要(Summary)
ciMethodData クラス用の補助クラス.

ciMethodData クラスから BranchData オブジェクト内のプロファイル情報を扱うためのラッパークラス.
1つの ciBranchData オブジェクトが 1つの BranchData オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciBranchData : public BranchData {
```

### 使われ方(Usage)
ciMethodData::data_at() 内で(のみ)生成されている.

### 内部構造(Internal structure)
何も定義されていない (なので中身はスーパークラスである BranchData と全く同じ).




### 詳細(Details)
See: [here](../doxygen/classciBranchData.html) for details

---
## <a name="noBsAxNdif" id="noBsAxNdif">ciArrayData</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

ciMethodData クラスから ArrayData オブジェクト内のプロファイル情報を扱うためのラッパークラスだと思われるが, 
ArrayData クラス自体が abstract class なので, このクラスは使用されない
(ArrayData 自体にはサブクラスがいるがこのクラスにはサブクラスはない.
 ArrayData のサブクラスに対応する ci* クラスは, 
 このクラスからではなく対応する ArrayData のサブクラス自身から派生している.)


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciArrayData : public ArrayData {
```

### 内部構造(Internal structure)
何も定義されていない (なので中身はスーパークラスである ArrayData と全く同じ).




### 詳細(Details)
See: [here](../doxygen/classciArrayData.html) for details

---
## <a name="noN6Ey380a" id="noN6Ey380a">ciMultiBranchData</a>

### 概要(Summary)
ciMethodData クラス用の補助クラス.

ciMethodData クラスから MultiBranchData オブジェクト内のプロファイル情報を扱うためのラッパークラス.
1つの ciMultiBranchData オブジェクトが 1つの MultiBranchData オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciMultiBranchData : public MultiBranchData {
```

### 使われ方(Usage)
ciMethodData::data_at() 内で(のみ)生成されている.

### 内部構造(Internal structure)
何も定義されていない (なので中身はスーパークラスである MultiBranchData と全く同じ).




### 詳細(Details)
See: [here](../doxygen/classciMultiBranchData.html) for details

---
## <a name="no_fq7wPTH" id="no_fq7wPTH">ciArgInfoData</a>

### 概要(Summary)
ciMethodData クラス用の補助クラス.

ciMethodData クラスから ArgInfoData オブジェクト内のプロファイル情報を扱うためのラッパークラス.
1つの ciArgInfoData オブジェクトが 1つの ArgInfoData オブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    class ciArgInfoData : public ArgInfoData {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* ciMethodData::data_at()
* ciMethodData::arg_info()
* ciMethodData::print_data_on()

### 内部構造(Internal structure)
何も定義されていない (なので中身はスーパークラスである ArgInfoData と全く同じ).




### 詳細(Details)
See: [here](../doxygen/classciArgInfoData.html) for details

---
## <a name="noArHbXFt0" id="noArHbXFt0">ciMethodData</a>

### 概要(Summary)
ciObject クラスの具象サブクラスの1つ. methodDataOopDesc 用の ciObject クラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciMethodData.hpp))
    // ciMethodData
    //
    // This class represents a methodDataOop in the HotSpot virtual
    // machine.
    
    class ciMethodData : public ciObject {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の箇所).

* 各 ciObjectFactory オブジェクトの _unloaded_methods フィールド
  
  unloaded/unfound なメソッドを表す ciMethod オブジェクトが格納されている.
  
  (正確には, このフィールドは ciMethod の GrowableArray を格納するフィールド.
  この中に, ciObjectFactory::get_unloaded_method() で生成された全ての ciMethod オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciObjectFactory::create_new_object()
  
  (ファクトリメソッド)

* ciObjectFactory::get_empty_methodData()

  (こちらは, methodDataOopDesc が存在しないメソッドのために, ダミーの(= 空の) ciMethodData オブジェクトを生成する関数)




### 詳細(Details)
See: [here](../doxygen/classciMethodData.html) for details

---
