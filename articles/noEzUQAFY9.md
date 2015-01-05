---
layout: default
title: ciConstantPoolCache クラス 
---
[Top](../index.html)

#### ciConstantPoolCache クラス 



---
## <a name="nokpS2qJRO" id="nokpS2qJRO">ciConstantPoolCache</a>

### 概要(Summary)
ciInstanceKlass クラス用の補助クラス.

一度参照されたフィールド (の index 番号) と ciField オブジェクトの対応付けをメモイズしておき, 
次回以降のフィールド参照処理を高速化するためのハッシュテーブル.

(なお, 名前が似ていて紛らわしいが constantPoolCacheOopDesc とは全く関係ない)

(なお, 現状では ciBytecodeStream クラスの処理からのみ利用されている模様)


```cpp
    ((cite: hotspot/src/share/vm/ci/ciConstantPoolCache.hpp))
    // ciConstantPoolCache
    //
    // The class caches indexed constant pool lookups.
    //
    // Usage note: this klass has nothing to do with constantPoolCacheOop.
    class ciConstantPoolCache : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ciInstanceKlass オブジェクトの _field_cache フィールドに(のみ)格納されている.
 
(ただし, ciConstantPoolCache オブジェクトの生成自体は実際に必要になるまで遅延されている)

#### 生成箇所(where its instances are created)
ciInstanceKlass::field_cache() 内で(のみ)生成されている (= 初めて使用される時まで生成が遅延されている).

#### 使用箇所(where its instances are used)
ciInstanceKlass::field_cache() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
ciBytecodeStream::get_field()
-&gt; ciEnv::get_field_by_index()
   -&gt; ciEnv::get_field_by_index_impl()
      -&gt; ciInstanceKlass::field_cache()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classciConstantPoolCache.html) for details

---
