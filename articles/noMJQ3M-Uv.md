---
layout: default
title: oopFactory クラス 
---
[Top](../index.html)

#### oopFactory クラス 



---
## <a name="no2H52_HE2" id="no2H52_HE2">oopFactory</a>

### 概要(Summary)
oopDesc クラス用のユーティリティ・クラス.

この中に oopDesc クラス (およびそのサブクラス) 用のファクトリメソッドを実装されている (つまり oopDesc 用のファクトリクラス)
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

ほとんどの (全ての?) oopDesc オブジェクトはこのクラスによって生成される模様 (#TODO).

(See: [here](no2114rPX.html) for details) (See: [here](no30267vB.html), [here](no28916Q0G.html), [here](no28916qPH.html) and [here](no289163ZN.html) for details)


```
    ((cite: hotspot/src/share/vm/memory/oopFactory.hpp))
    // oopFactory is a class used for creating new objects.
```


```
    ((cite: hotspot/src/share/vm/memory/oopFactory.hpp))
    class oopFactory: AllStatic {
```

### 内部構造(Internal structure)
各 oopDesc クラスに対応する以下のようなファクトリメソッド (new_*() メソッド) を提供している.


```
    ((cite: hotspot/src/share/vm/memory/oopFactory.hpp))
      // Basic type leaf array allocation
      static typeArrayOop    new_boolArray  (int length, TRAPS) { return typeArrayKlass::cast(Universe::boolArrayKlassObj  ())->allocate(length, CHECK_NULL); }
      static typeArrayOop    new_charArray  (int length, TRAPS) { return typeArrayKlass::cast(Universe::charArrayKlassObj  ())->allocate(length, CHECK_NULL); }
      static typeArrayOop    new_singleArray(int length, TRAPS) { return typeArrayKlass::cast(Universe::singleArrayKlassObj())->allocate(length, CHECK_NULL); }
      static typeArrayOop    new_doubleArray(int length, TRAPS) { return typeArrayKlass::cast(Universe::doubleArrayKlassObj())->allocate(length, CHECK_NULL); }
      static typeArrayOop    new_byteArray  (int length, TRAPS) { return typeArrayKlass::cast(Universe::byteArrayKlassObj  ())->allocate(length, CHECK_NULL); }
      static typeArrayOop    new_shortArray (int length, TRAPS) { return typeArrayKlass::cast(Universe::shortArrayKlassObj ())->allocate(length, CHECK_NULL); }
      static typeArrayOop    new_intArray   (int length, TRAPS) { return typeArrayKlass::cast(Universe::intArrayKlassObj   ())->allocate(length, CHECK_NULL); }
      static typeArrayOop    new_longArray  (int length, TRAPS) { return typeArrayKlass::cast(Universe::longArrayKlassObj  ())->allocate(length, CHECK_NULL); }
    
      // create java.lang.Object[]
      static objArrayOop     new_objectArray(int length, TRAPS)  {
    ...
    
      static typeArrayOop    new_charArray           (const char* utf8_str,  TRAPS);
      static typeArrayOop    new_permanent_charArray (int length, TRAPS);
      static typeArrayOop    new_permanent_byteArray (int length, TRAPS);  // used for class file structures
      static typeArrayOop    new_permanent_shortArray(int length, TRAPS);  // used for class file structures
      static typeArrayOop    new_permanent_intArray  (int length, TRAPS);  // used for class file structures
    
      static typeArrayOop    new_typeArray(BasicType type, int length, TRAPS);
    
    ...
      static constantPoolOop      new_constantPool     (int length,
    ...
      static constantPoolCacheOop new_constantPoolCache(int length,
    ...
      static klassOop        new_instanceKlass(Symbol* name,
    ...
      static constMethodOop  new_constMethod(int byte_code_size,
    ...
      static methodOop       new_method(int byte_code_size,
    ...
      static methodDataOop   new_methodData(methodHandle method, TRAPS);
    ...
      static objArrayOop     new_system_objArray(int length, TRAPS);
    ...
      static objArrayOop     new_objArray(klassOop klass, int length, TRAPS);
    ...
      static compiledICHolderOop new_compiledICHolder(methodHandle method, KlassHandle klass, TRAPS);
```




### 詳細(Details)
See: [here](../doxygen/classoopFactory.html) for details

---
