---
layout: default
title: StackValue クラス 
---
[Top](../index.html)

#### StackValue クラス 



---
## <a name="noeGtoD4NP" id="noeGtoD4NP">StackValue</a>

### 概要(Summary)
StackValueCollection クラス用の補助クラス.

スタックフレーム上の値を参照する処理で使用される一時オブジェクト(ResourceObjクラス).
スタックフレーム上にある値を表す.

1つの StackValue オブジェクトは以下のどれかに対応する.

* あるスタックフレーム上の 1つの引数, または 1つの局所変数
* あるスタックフレーム上のオペランドスタック内にある 1つの値


```cpp
    ((cite: hotspot/src/share/vm/runtime/stackValue.hpp))
    class StackValue : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 StackValueCollection オブジェクトの _values フィールドに(のみ)格納されている.

(正確には, このフィールドは StackValue の GrowableArray を格納するフィールド.
この中に, その StackValueCollection 用の全ての StackValue オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ただし, ResourceObjクラスなので一時的なオブジェクト).

* StackValue::create_stack_value()
* interpretedVFrame::locals()
* interpretedVFrame::expressions()
* vframeArrayElement::fill_in()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
* 脱最適化処理 (Deoptimization 処理)

  Deoptimization::fetch_unroll_info_helper()
  -&gt; Deoptimization::reassign_fields()
     -&gt; instanceKlass::do_nonstatic_fields()
        -&gt; FieldReassigner::do_field()
           -&gt; StackValue::create_stack_value()
     -&gt; Deoptimization::reassign_type_array_elements()
        -&gt; StackValue::create_stack_value()
     -&gt; Deoptimization::reassign_object_array_elements()
        -&gt; StackValue::create_stack_value()
  -&gt; Deoptimization::create_vframeArray()
     -&gt; vframeArray::allocate()
        -&gt; vframeArray::fill_in()
           -&gt; vframeArrayElement::fill_in()

* interpretedVFrame/compiledVFrame 関係の処理 (#TODO)

  interpretedVFrame::locals()

  interpretedVFrame::expressions()
  
  compiledVFrame::locals()
  -&gt; compiledVFrame::create_stack_value()
     -&gt; StackValue::create_stack_value()
  
  compiledVFrame::expressions()
  -&gt; compiledVFrame::create_stack_value()
     -&gt; StackValue::create_stack_value()
  
  compiledVFrame::monitors()
  -&gt; compiledVFrame::create_stack_value()
     -&gt; StackValue::create_stack_value()
</pre></div>

なお, StackValueCollection::_values フィールドの GrowableArray 自体は 
StackValueCollection::StackValueCollection() 内で(のみ)確保されている. 

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.


```cpp
    ((cite: hotspot/src/share/vm/runtime/stackValue.hpp))
      BasicType _type;
      intptr_t  _i; // Blank java stack slot value
      Handle    _o; // Java stack slot value interpreted as a Handle
```




### 詳細(Details)
See: [here](../doxygen/classStackValue.html) for details

---
