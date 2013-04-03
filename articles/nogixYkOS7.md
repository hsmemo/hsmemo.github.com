---
layout: default
title: oop (oop, instanceOop, methodOop, constMethodOop, methodDataOop, arrayOop, objArrayOop, typeArrayOop, constantPoolOop, constantPoolCacheOop, klassOop, markOop, compiledICHolderOop)
---
[Top](../index.html)

#### oop (oop, instanceOop, methodOop, constMethodOop, methodDataOop, arrayOop, objArrayOop, typeArrayOop, constantPoolOop, constantPoolCacheOop, klassOop, markOop, compiledICHolderOop)

### 概要(Summary)
各 oopDesc クラスは, 実際の使用箇所では "Desc" を外した名称で使われることが多い
(e.g. oopDesc -> oop, instanceOopDesc -> instanceOop, etc).

これらは通常のビルド時には単なる別名(typedef)に過ぎない (対応する oopDesc クラスへのポインタ型).


```
    ((cite: hotspot/src/share/vm/oops/oopsHierarchy.hpp))
    #ifndef CHECK_UNHANDLED_OOPS
    
    typedef class oopDesc*                            oop;
    typedef class   instanceOopDesc*            instanceOop;
    typedef class   methodOopDesc*                    methodOop;
    typedef class   constMethodOopDesc*            constMethodOop;
    typedef class   methodDataOopDesc*            methodDataOop;
    typedef class   arrayOopDesc*                    arrayOop;
    typedef class     objArrayOopDesc*            objArrayOop;
    typedef class     typeArrayOopDesc*            typeArrayOop;
    typedef class   constantPoolOopDesc*            constantPoolOop;
    typedef class   constantPoolCacheOopDesc*   constantPoolCacheOop;
    typedef class   klassOopDesc*                    klassOop;
    typedef class   markOopDesc*                    markOop;
    typedef class   compiledICHolderOopDesc*    compiledICHolderOop;
```

ただし, デバッグ時(開発時)には元の *OopDesc クラスのラッパークラスとすることができる.

このラッパークラスを使うと「その oop が適切に Handle 化されているかどうか」の実行時チェック(Unhandled Oops Check)を自動的に行うことができる
(See: [here](no2935rfO.html) for details).


### クラス一覧(class list)

  * [oop](#noGwHroyv4)
  * [instanceOop, methodOop, constMethodOop, methodDataOop, arrayOop, objArrayOop, typeArrayOop, constantPoolOop, constantPoolCacheOop, klassOop, compiledICHolderOop](#no6tV7qG16)
  * [markOop](#noSv1zvWLZ)


---
## <a name="noGwHroyv4" id="noGwHroyv4">oop</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef CHECK_UNHANDLED_OOPS 時以外にはクラスとしては定義されず単なる typedef になる).

実行時に「適切に Handle 化されているか (= Handle 化されずに Safepoint を迎えていないか)」を自動的に検査してくれる
(See: [here](no2935rfO.html) for details).

その他の点では "oopDesc*" (oopDesc へのポインタ) とほぼ同じ感覚で使用できる
(そのために各種の演算子がオーバーライドされている).


```
    ((cite: hotspot/src/share/vm/oops/oopsHierarchy.hpp))
    class oop {
```




### 詳細(Details)
See: [here](../doxygen/classoop.html) for details

---
## <a name="no6tV7qG16" id="no6tV7qG16">instanceOop, methodOop, constMethodOop, methodDataOop, arrayOop, objArrayOop, typeArrayOop, constantPoolOop, constantPoolCacheOop, klassOop, compiledICHolderOop</a>

### 概要(Summary)
oop クラスのサブクラス群.
oop クラスと同じく, これらも対応する oopDesc クラスへのポインタ型とほぼ同じ感覚で使用できる.

なお, これらのクラスは
DEF_OOP というマクロを使って間接的に定義されている.


```
    ((cite: hotspot/src/share/vm/oops/oopsHierarchy.hpp))
    DEF_OOP(instance);
    DEF_OOP(method);
    DEF_OOP(methodData);
    DEF_OOP(array);
    DEF_OOP(constMethod);
    DEF_OOP(constantPool);
    DEF_OOP(constantPoolCache);
    DEF_OOP(objArray);
    DEF_OOP(typeArray);
    DEF_OOP(klass);
    DEF_OOP(compiledICHolder);
```

### 備考(Notes)
DEF_OOP() マクロの定義は以下の通り.


```
    ((cite: hotspot/src/share/vm/oops/oopsHierarchy.hpp))
    #define DEF_OOP(type)                                                      \
       class type##OopDesc;                                                    \
       class type##Oop : public oop {                                          \
         public:                                                               \
           type##Oop() : oop() {}                                              \
           type##Oop(const volatile oop& o) : oop(o) {}                        \
           type##Oop(const void* p) : oop(p) {}                                \
           operator type##OopDesc* () const { return (type##OopDesc*)obj(); }  \
           type##OopDesc* operator->() const {                                 \
                return (type##OopDesc*)obj();                                  \
           }                                                                   \
       };                                                                      \
```



### 詳細(Details)
See: [here](../doxygen/classinstanceOop, methodOop, constMethodOop, methodDataOop, arrayOop, objArrayOop, typeArrayOop, constantPoolOop, constantPoolCacheOop, klassOop, compiledICHolderOop.html) for details

---
## <a name="noSv1zvWLZ" id="noSv1zvWLZ">markOop</a>

### 概要(Summary)
markOopDesc へのポインタを表す型.

markOop については, (ラッパークラス化は難しいので(?))
CHECK_UNHANDLED_OOPS に関わらず単なる別名として定義される.


```
    ((cite: hotspot/src/share/vm/oops/oopsHierarchy.hpp))
    typedef class   markOopDesc*                markOop;
```




### 詳細(Details)
See: [here](../doxygen/classmarkOop.html) for details

---
