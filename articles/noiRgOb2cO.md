---
layout: default
title: ciObjectFactory クラス 
---
[Top](../index.html)

#### ciObjectFactory クラス 



---
## <a name="noDiYDfZLg" id="noDiYDfZLg">ciObjectFactory</a>

### 概要(Summary)
ciEnv クラス内で使用される補助クラス.

ciObject クラス (およびそのサブクラス) 用のファクトリメソッドを実装したクラス(ファクトリクラス).
ほとんどの ci* オブジェクトはこのクラスによって生成される.

なお, 一度生成した ciObject オブジェクトはメモイズしてあり, 同じ oop に対しては同一の ci* オブジェクトを返すようになっている.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciObjectFactory.hpp))
    // ciObjectFactory
    //
    // This class handles requests for the creation of new instances
    // of ciObject and its subclasses.  It contains a caching mechanism
    // which ensures that for each oop, at most one ciObject is created.
    // This invariant allows efficient implementation of ciObject.
    class ciObjectFactory : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ciEnv オブジェクトの _factory フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciEnv::ciEnv(CompileTask* task, int system_dictionary_modification_counter)
* ciEnv::ciEnv(Arena* arena)

### 内部構造(Internal structure)
一度生成した ciObject オブジェクトは以下の GrowableArray にメモイズしている
(なおコメントによると, ソートして格納しているため取り出す際には二分探索できて高速だが挿入は遅い, とのこと).

* ciObjectFactory::_shared_ci_objects


```cpp
    ((cite: hotspot/src/share/vm/ci/ciObjectFactory.cpp))
    // Implementation note: the oop->ciObject mapping is represented as
    // a table stored in an array.  Even though objects are moved
    // by the garbage collector, the compactor preserves their relative
    // order; address comparison of oops (in perm space) is safe so long
    // as we prohibit GC during our comparisons.  We currently use binary
    // search to find the oop in the table, and inserting a new oop
    // into the table may be costly.  If this cost ends up being
    // problematic the underlying data structure can be switched to some
    // sort of balanced binary tree.
    
    GrowableArray<ciObject*>* ciObjectFactory::_shared_ci_objects = NULL;
```

### 備考(Notes)
ciObjectFactory::create_new_object() が ciObject (とそのサブクラス) 用のファクトリメソッドになっている.
このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
ciEnv::get_object()
-&gt; ciObjectFactory::get()
   -&gt; ciObjectFactory::create_new_object()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classciObjectFactory.html) for details

---
