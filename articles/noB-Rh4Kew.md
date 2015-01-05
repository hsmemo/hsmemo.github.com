---
layout: default
title: Reflection クラス関連のクラス (KlassStream, MethodStream, FieldStream, FilteredField, FilteredFieldsMap, FilteredFieldStream)
---
[Top](../index.html)

#### Reflection クラス関連のクラス (KlassStream, MethodStream, FieldStream, FilteredField, FilteredFieldsMap, FilteredFieldStream)

これらは, Java のクラス内のメソッド情報やフィールド情報にアクセスするためのユーティリティ・クラス.


### クラス一覧(class list)

  * [KlassStream](#nof4ilc1M9)
  * [MethodStream](#noOtmrZv_M)
  * [FieldStream](#noWI_xw0Ex)
  * [FilteredFieldStream](#noPz841EYf)
  * [FilteredFieldsMap](#noDYSoijuH)
  * [FilteredField](#nobwQDjdGz)


---
## <a name="nof4ilc1M9" id="nof4ilc1M9">KlassStream</a>

### 概要(Summary)
指定したクラスを起点として, 
そのスーパークラスやスーパーインターフェースを辿っていくためのイテレータクラス(ValueObjクラス) (の基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/reflectionUtils.hpp))
    // A KlassStream is an abstract stream for streaming over self, superclasses
    // and (super)interfaces. Streaming is done in reverse order (subclasses first,
    // interfaces last).
    //
    //    for (KlassStream st(k, false, false); !st.eos(); st.next()) {
    //      klassOop k = st.klass();
    //      ...
    //    }
    
    class KlassStream VALUE_OBJ_CLASS_SPEC {
```




### 詳細(Details)
See: [here](../doxygen/classKlassStream.html) for details

---
## <a name="noOtmrZv_M" id="noOtmrZv_M">MethodStream</a>

### 概要(Summary)
指定したクラスを起点として, 
そのスーパークラスやスーパーインターフェースのメソッドを辿っていくためのイテレータクラス(ValueObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/reflectionUtils.hpp))
    // A MethodStream streams over all methods in a class, superclasses and (super)interfaces.
    // Streaming is done in reverse order (subclasses first, methods in reverse order)
    // Usage:
    //
    //    for (MethodStream st(k, false, false); !st.eos(); st.next()) {
    //      methodOop m = st.method();
    //      ...
    //    }
    
    class MethodStream : public KlassStream {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* MethodHandles::find_MemberNames()
* Reflection::reflect_method()
* Reflection::reflect_methods()
* Reflection::reflect_constructor()
* Reflection::reflect_constructors()




### 詳細(Details)
See: [here](../doxygen/classMethodStream.html) for details

---
## <a name="noWI_xw0Ex" id="noWI_xw0Ex">FieldStream</a>

### 概要(Summary)
指定したクラスを起点として, 
そのスーパークラスやスーパーインターフェースのフィールドを辿っていくためのイテレータクラス(ValueObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/reflectionUtils.hpp))
    // A FieldStream streams over all fields in a class, superclasses and (super)interfaces.
    // Streaming is done in reverse order (subclasses first, fields in reverse order)
    // Usage:
    //
    //    for (FieldStream st(k, false, false); !st.eos(); st.next()) {
    //      Symbol* field_name = st.name();
    //      ...
    //    }
    
    
    class FieldStream : public KlassStream {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* MethodHandles::find_MemberNames()
* Reflection::reflect_field()
* Reflection::reflect_fields()
* DumperSupport::instance_size()
* DumperSupport::dump_static_fields()
* DumperSupport::dump_instance_fields()
* DumperSupport::dump_instance_field_descriptors()




### 詳細(Details)
See: [here](../doxygen/classFieldStream.html) for details

---
## <a name="noPz841EYf" id="noPz841EYf">FilteredFieldStream</a>

### 概要(Summary)
特殊な FieldStream クラス.

FieldStream と異なり, HotSpot の内部処理用のフィールドはスキップする.


```cpp
    ((cite: hotspot/src/share/vm/runtime/reflectionUtils.hpp))
    // A FilteredFieldStream streams over all fields in a class, superclasses and
    // (super)interfaces. Streaming is done in reverse order (subclasses first,
    // fields in reverse order)
    //
    // Usage:
    //
    //    for (FilteredFieldStream st(k, false, false); !st.eos(); st.next()) {
    //      Symbol* field_name = st.name();
    //      ...
    //    }
    
    class FilteredFieldStream : public FieldStream {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JvmtiEnv::GetClassFields()
* ClassFieldMap::create_map_of_static_fields()
* ClassFieldMap::create_map_of_instance_fields()

### 内部構造(Internal structure)
スキップするかどうかの判定は FilteredFieldsMap クラスが行っている. 

(より正確に言うと FilteredFieldsMap クラスの FilteredFieldsMap::is_filtered_field() メソッドが行っている.
 FilteredFieldsMap::is_filtered_field() が true を返す場合は, 
 そのフィールドをスキップして次のフィールドに進む)

#### 参考(for your information): FilteredFieldStream::next()
See: [here](no1904jvn.html) for details



### 詳細(Details)
See: [here](../doxygen/classFilteredFieldStream.html) for details

---
## <a name="noDYSoijuH" id="noDYSoijuH">FilteredFieldsMap</a>

### 概要(Summary)
FilteredFieldStream クラス用の補助クラス.

FilteredFieldStream の処理でスキップしたいフィールドを管理するためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/share/vm/runtime/reflectionUtils.hpp))
    class FilteredFieldsMap : AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

<div class="flow-abst"><pre>
* このクラスの初期化処理

  javaClasses_init()
  -&gt; FilteredFieldsMap::initialize()

* FilteredFieldStream の処理

  FilteredFieldStream::FilteredFieldStream()
  -&gt; FilteredFieldsMap::filtered_fields_count()

  FilteredFieldStream::next()
  -&gt; FilteredFieldsMap::filtered_fields_count()

  FilteredFieldStream::next()
  -&gt; FilteredFieldsMap::is_filtered_field()

* GC 処理 (より正確に言うと GC 時に FilteredFieldsMap 内に格納しているポインタを辿る処理)

  SystemDictionary::preloaded_oops_do()
  -&gt; FilteredFieldsMap::klasses_oops_do()
</pre></div>

### 内部構造(Internal structure)
実際のスキップ対象の情報は FilteredField オブジェクトが格納している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/reflectionUtils.hpp))
     private:
      static GrowableArray<FilteredField *> *_filtered_fields;
```

### 備考(Notes)
現在は, 以下の三つのフィールド(のみ)をスキップ対象としている.

  * java.lang.Throwable.backtrace
    
    このフィールドは, ソースコード上では Object 型になっているが,
    実際には Java のオブジェクトではなく HotSpot の内部的なデータ構造が納められているため除外されている
    [(参考)](http://bugs.sun.com/view_bug.do?bug_id=4496456).

  * sun.reflect.ConstantPool.constantPoolOop  (JDK 6 以降の場合のみ)
    
    除外されている理由は恐らく java.lang.Throwable.backtrace フィールドと同様 (#TODO).

  * sun.reflect.UnsafeStaticFieldAccessorImpl.base  (JDK 6 以降の場合のみ)
    
    除外されている理由は恐らく java.lang.Throwable.backtrace フィールドと同様 (#TODO).


```cpp
    ((cite: hotspot/src/share/vm/runtime/reflectionUtils.cpp))
    void FilteredFieldsMap::initialize() {
    ...
      offset = java_lang_Throwable::get_backtrace_offset();
      _filtered_fields->append(new FilteredField(SystemDictionary::Throwable_klass(), offset));
      // The latest version of vm may be used with old jdk.
      if (JDK_Version::is_gte_jdk16x_version()) {
        // The following class fields do not exist in
        // previous version of jdk.
        offset = sun_reflect_ConstantPool::cp_oop_offset();
        _filtered_fields->append(new FilteredField(SystemDictionary::reflect_ConstantPool_klass(), offset));
        offset = sun_reflect_UnsafeStaticFieldAccessorImpl::base_offset();
        _filtered_fields->append(new FilteredField(SystemDictionary::reflect_UnsafeStaticFieldAccessorImpl_klass(), offset));
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classFilteredFieldsMap.html) for details

---
## <a name="nobwQDjdGz" id="nobwQDjdGz">FilteredField</a>

### 概要(Summary)
FilteredFieldsMap クラス内で使用される補助クラス.

FilteredFieldStream の処理でスキップしたいフィールドを表すクラス.
1つの FilteredField オブジェクトが 1つのフィールドに対応する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/reflectionUtils.hpp))
    class FilteredField {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
FilteredFieldsMap クラスの _filtered_fields フィールド (static フィールド) に(のみ)格納されている.

(正確には, このフィールドは FilteredField の GrowableArray を格納するフィールド.
この中に, 全ての FilteredField オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
FilteredFieldsMap::initialize() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* FilteredFieldsMap::is_filtered_field()
* FilteredFieldsMap::filtered_fields_count()
* FilteredFieldsMap::klasses_oops_do()

### 内部構造(Internal structure)
定義されているフィールドは以下の通り
(そして, メソッドはこれらのフィールドへのアクセサメソッドのみ).

(それぞれ, フィルタリング対象のクラス, およびそのフィールドのオフセットを表す)


```cpp
    ((cite: hotspot/src/share/vm/runtime/reflectionUtils.hpp))
      klassOop _klass;
      int      _field_offset;
```




### 詳細(Details)
See: [here](../doxygen/classFilteredField.html) for details

---
