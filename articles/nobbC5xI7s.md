---
layout: default
title: ciField クラス 
---
[Top](../index.html)

#### ciField クラス 



---
## <a name="no8SrN7trz" id="no8SrN7trz">ciField</a>

### 概要(Summary)
JIT Compiler からクラスのフィールド情報にアクセスするための一時オブジェクト(ResourceObjクラス).
1つの ciField オブジェクトが 1つのフィールドに対応する.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciField.hpp))
    // ciField
    //
    // This class represents the result of a field lookup in the VM.
    // The lookup may not succeed, in which case the information in
    // the ciField will be incomplete.
    class ciField : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の箇所).

* 各 ciInstanceKlass オブジェクトの _nonstatic_fields フィールド
  
  ciInstanceKlass::get_field_by_offset() 用のフィールド.
  
  (正確には, このフィールドは ciField の GrowableArray を格納するフィールド.
  この中に, そのクラスの static ではないフィールドに対応する全ての ciField オブジェクトが格納されている)
  
  (ただし, この GrowableArray の生成や初期化自体は実際に必要になるまで遅延されている.
   (See: ciInstanceKlass::compute_nonstatic_fields()))

* 各 ciInstanceKlass オブジェクトの _non_static_fields フィールド
  
  ?? (このフィールドは使用箇所が見当たらないような...)

  ciInstanceKlass::non_static_fields() 用のフィールド
  (ただし, 肝心の ciInstanceKlass::non_static_fields() 自体が使われていないような... #TODO)
  
  (正確には, このフィールドは ciField の GrowableArray を格納するフィールド.
  この中に, そのクラスの static ではないフィールドに対応する全ての ciField オブジェクトが格納されている)
  
  (ただし, この GrowableArray の生成や初期化自体は実際に必要になるまで遅延されている.
   (See: ciInstanceKlass::non_static_fields()))

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciEnv::get_field_by_index_impl()
* ciInstanceKlass::compute_nonstatic_fields()
* ciInstanceKlass::compute_nonstatic_fields_impl()
* ciInstanceKlass::get_field_by_offset()
* ciInstanceKlass::get_field_by_name()
* NonStaticFieldFiller::do_field()

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
ciBytecodeStream::get_field()
-&gt; ciEnv::get_field_by_index()
   -&gt; ciEnv::get_field_by_index_impl()

ciInstanceKlass::nof_nonstatic_fields()
-&gt; ciInstanceKlass::compute_nonstatic_fields()
   -&gt; ciInstanceKlass::compute_nonstatic_fields_impl()

ciObjectFactory::init_shared_objects()
-&gt; ciInstanceKlass::compute_nonstatic_fields()
   -&gt; (同上)

ciInstanceKlass::get_canonical_holder()
-&gt; ciInstanceKlass::nof_nonstatic_fields()
   -&gt; (同上)

ciInstanceKlass::get_field_by_offset()
-&gt; ciInstanceKlass::nof_nonstatic_fields()
   -&gt; (同上)

ciInstanceKlass::get_field_by_name()

ciInstanceKlass::non_static_fields()
-&gt; instanceKlass::do_nonstatic_fields()
   -&gt; NonStaticFieldFiller::do_field()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classciField.html) for details

---
