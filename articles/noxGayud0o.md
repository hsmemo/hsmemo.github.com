---
layout: default
title: Handle クラス関連のクラス (Handle, KlassHandle,  instanceHandle, methodHandle, constMethodHandle, methodDataHandle, arrayHandle, constantPoolHandle, constantPoolCacheHandle, objArrayHandle, typeArrayHandle,  instanceKlassHandle, methodKlassHandle, constMethodKlassHandle, klassKlassHandle, arrayKlassKlassHandle, objArrayKlassKlassHandle, typeArrayKlassKlassHandle, arrayKlassHandle, typeArrayKlassHandle, objArrayKlassHandle, constantPoolKlassHandle, constantPoolCacheKlassHandle,  HandleArea, HandleMark, NoHandleMark, ResetNoHandleMark)
---
[Top](../index.html)

#### Handle クラス関連のクラス (Handle, KlassHandle,  instanceHandle, methodHandle, constMethodHandle, methodDataHandle, arrayHandle, constantPoolHandle, constantPoolCacheHandle, objArrayHandle, typeArrayHandle,  instanceKlassHandle, methodKlassHandle, constMethodKlassHandle, klassKlassHandle, arrayKlassKlassHandle, objArrayKlassKlassHandle, typeArrayKlassKlassHandle, arrayKlassHandle, typeArrayKlassHandle, objArrayKlassHandle, constantPoolKlassHandle, constantPoolCacheKlassHandle,  HandleArea, HandleMark, NoHandleMark, ResetNoHandleMark)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, GC によるポインタ移動の影響を HotSpot 内の他の部分から隠蔽するためのクラス (See: [here](no3718kvd.html) for details).

### 概要(Summary)
Java のオブジェクト(oop) は GC 時に移動されることがある.
このため HotSpot 内で oop のポインタを直接触っていると, 
いつの間にか不正なポインタ(dangling pointer)になっている恐れがあり面倒臭い.

そこで, HotSpot 内で oop のポインタを操作する場合は "Handle" というオブジェクトを介して関節参照している.

全ての Handle は GC 時の調査対象になり, オブジェクトが移動した場合は Handle 内のポインタも新しいものに書き換わる.
このため, Handle を使うコードからは GC の影響を無視することができる.

(以下は Handle の使用例)

      oop obj = ...;
      Handle h1(obj);              // allocate new handle
      Handle h2(thread, obj);      // faster allocation when current thread is known
      Handle h3;                   // declare handle only, no allocation occurs
      ...
      h3 = h1;                     // make h3 refer to same indirection as h1
      oop obj2 = h2();             // get handle value
      h1->print();                 // invoking operation on oop

なお, Handle には各 oop クラスに対応するサブクラスが用意されている
(コメントによると, クラスに応じた追加情報が付けられるし無駄なキャストも減る, とのこと).

      oop           Handle
      methodOop     methodHandle
      instanceOop   instanceHandle

さらに, klassOop については Klass クラスごとに Handle が用意されている
(コメントによると, キャストなしで klass_part にアクセス可能にしてコードを書くのが楽にするため, とのこと).

      klassOop      Klass           KlassHandle
      klassOop      methodKlass     methodKlassHandle
      klassOop      instanceKlass   instanceKlassHandle


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    //------------------------------------------------------------------------------------------------------------------------
    // In order to preserve oops during garbage collection, they should be
    // allocated and passed around via Handles within the VM. A handle is
    // simply an extra indirection allocated in a thread local handle area.
    //
    // A handle is a ValueObj, so it can be passed around as a value, can
    // be used as a parameter w/o using &-passing, and can be returned as a
    // return value.
    //
    // oop parameters and return types should be Handles whenever feasible.
    //
    // Handles are declared in a straight-forward manner, e.g.
    //
    //   oop obj = ...;
    //   Handle h1(obj);              // allocate new handle
    //   Handle h2(thread, obj);      // faster allocation when current thread is known
    //   Handle h3;                   // declare handle only, no allocation occurs
    //   ...
    //   h3 = h1;                     // make h3 refer to same indirection as h1
    //   oop obj2 = h2();             // get handle value
    //   h1->print();                 // invoking operation on oop
    //
    // Handles are specialized for different oop types to provide extra type
    // information and avoid unnecessary casting. For each oop type xxxOop
    // there is a corresponding handle called xxxHandle, e.g.
    //
    //   oop           Handle
    //   methodOop     methodHandle
    //   instanceOop   instanceHandle
    //
    // For klassOops, it is often useful to model the Klass hierarchy in order
    // to get access to the klass_part without casting. For each xxxKlass there
    // is a corresponding handle called xxxKlassHandle, e.g.
    //
    //   klassOop      Klass           KlassHandle
    //   klassOop      methodKlass     methodKlassHandle
    //   klassOop      instanceKlass   instanceKlassHandle
    //
```

なお, Handle が必要とするデータは JavaThread 内にある HandleArea という Arena 上に確保されることになっている.
(Handle オブジェクト自体はその領域を指すポインタだけを保持する. ポインタ1つなので ValueObj として扱われている)


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.inline.hpp))
    inline Handle::Handle(oop obj) {
      if (obj == NULL) {
        _handle = NULL;
      } else {
        _handle = Thread::current()->handle_area()->allocate_handle(obj);
      }
    }
```

そして, HandleArea 内のメモリ管理は, HandleMark というクラスにより, ソースコード上のスコープに合わせて行うことができるようになっている
(参考: [Region-based memory management](http://en.wikipedia.org/wiki/Region-based_memory_management)).

(以下は使用例: スコープを抜けて HandleMark のデストラクタが呼ばれた段階で, そのスコープ内で確保した Handle が全て解放される)


```cpp
    ((cite: hotspot/src/share/vm/runtime/javaCalls.cpp))


        { HandleMark hm(thread);  // HandleMark used by HandleMarkCleaner
    
    ...
        }
```



### クラス一覧(class list)

  * [Handle](#no-CdELD9N)
  * [KlassHandle](#noVhsXnDaq)
  * [instanceHandle](#nookS1WrKl)
  * [methodHandle](#nouKaJSzAt)
  * [constMethodHandle](#noenlaSCpQ)
  * [methodDataHandle](#novf_MBLhI)
  * [arrayHandle](#noUqVnukrL)
  * [constantPoolHandle](#nocDaz_h_h)
  * [constantPoolCacheHandle](#noqOh1LCt8)
  * [objArrayHandle](#noNLlE98CP)
  * [typeArrayHandle](#no2FjRWV1z)
  * [instanceKlassHandle](#no51I7dDH4)
  * [methodKlassHandle](#noJRUBjyHh)
  * [constMethodKlassHandle](#noQKBjOc8g)
  * [klassKlassHandle](#noOeEP91Dp)
  * [arrayKlassKlassHandle](#no9XLAfRXA)
  * [objArrayKlassKlassHandle](#nohfbH-q3Q)
  * [typeArrayKlassKlassHandle](#noLcM0vxpc)
  * [arrayKlassHandle](#noiw8Uzf-t)
  * [typeArrayKlassHandle](#no3Ri2OaVj)
  * [objArrayKlassHandle](#noY_W0TNCR)
  * [constantPoolKlassHandle](#no45oLBOrA)
  * [constantPoolCacheKlassHandle](#noRb2qu9rU)
  * [HandleArea](#no8KqXvHt4)
  * [HandleMark](#no7QqAmO2A)
  * [NoHandleMark](#noetnh7N_B)
  * [ResetNoHandleMark](#noC_9tJZH_)


---
## <a name="no-CdELD9N" id="no-CdELD9N">Handle</a>

### 概要(Summary)
GC によるポインタ移動の影響を HotSpot 内の他の部分から隠蔽するためのクラス.
HotSpot 内で Java のオブジェクト (oop) を操作する場合は Handle 経由で行う (See: [here](no3718kvd.html) for details).

なお Handle 自体は ValueObj であり, メソッドの引数や返り値として使うこともできる.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    //------------------------------------------------------------------------------------------------------------------------
    // Base class for all handles. Provides overloading of frequently
    // used operators for ease of use.
    
    class Handle VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
Handle が必要とするデータは HandleArea という Arena 上に確保される
(Handle オブジェクト自体はその領域を指すポインタだけを保持する. このため ValueObj クラスになっている).




### 詳細(Details)
See: [here](../doxygen/classHandle.html) for details

---
## <a name="noVhsXnDaq" id="noVhsXnDaq">KlassHandle</a>

### 概要(Summary)
klassOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    //------------------------------------------------------------------------------------------------------------------------
    // Base class for Handles containing klassOops. Provides overloading of frequently
    // used operators for ease of use and typed access to the Klass part.
    class KlassHandle: public Handle {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 備考(Notes)
Klass の種別が分かっている場合は KlassHandle の各サブクラスも利用可能.




### 詳細(Details)
See: [here](../doxygen/classKlassHandle.html) for details

---
## <a name="nookS1WrKl" id="nookS1WrKl">instanceHandle</a>

### 概要(Summary)
instanceOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_HANDLE(instance         , is_instance         )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific Handles for different oop types
    #define DEF_HANDLE(type, is_a)                   \
      class type##Handle;                            \
      class type##Handle: public Handle {            \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classinstanceHandle.html) for details

---
## <a name="nouKaJSzAt" id="nouKaJSzAt">methodHandle</a>

### 概要(Summary)
methodOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_HANDLE(method           , is_method           )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific Handles for different oop types
    #define DEF_HANDLE(type, is_a)                   \
      class type##Handle;                            \
      class type##Handle: public Handle {            \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 備考(Notes)
名前が似ていて紛らわしいが java.lang.invoke.MethodHandle とは全く関係ない.




### 詳細(Details)
See: [here](../doxygen/classmethodHandle.html) for details

---
## <a name="noenlaSCpQ" id="noenlaSCpQ">constMethodHandle</a>

### 概要(Summary)
constMethodOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_HANDLE(constMethod      , is_constMethod      )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific Handles for different oop types
    #define DEF_HANDLE(type, is_a)                   \
      class type##Handle;                            \
      class type##Handle: public Handle {            \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classconstMethodHandle.html) for details

---
## <a name="novf_MBLhI" id="novf_MBLhI">methodDataHandle</a>

### 概要(Summary)
methodDataOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_HANDLE(methodData       , is_methodData       )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific Handles for different oop types
    #define DEF_HANDLE(type, is_a)                   \
      class type##Handle;                            \
      class type##Handle: public Handle {            \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classmethodDataHandle.html) for details

---
## <a name="noUqVnukrL" id="noUqVnukrL">arrayHandle</a>

### 概要(Summary)
arrayOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_HANDLE(array            , is_array            )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific Handles for different oop types
    #define DEF_HANDLE(type, is_a)                   \
      class type##Handle;                            \
      class type##Handle: public Handle {            \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classarrayHandle.html) for details

---
## <a name="nocDaz_h_h" id="nocDaz_h_h">constantPoolHandle</a>

### 概要(Summary)
constantPoolOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_HANDLE(constantPool     , is_constantPool     )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific Handles for different oop types
    #define DEF_HANDLE(type, is_a)                   \
      class type##Handle;                            \
      class type##Handle: public Handle {            \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classconstantPoolHandle.html) for details

---
## <a name="noqOh1LCt8" id="noqOh1LCt8">constantPoolCacheHandle</a>

### 概要(Summary)
constantPoolCacheOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_HANDLE(constantPoolCache, is_constantPoolCache)
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific Handles for different oop types
    #define DEF_HANDLE(type, is_a)                   \
      class type##Handle;                            \
      class type##Handle: public Handle {            \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classconstantPoolCacheHandle.html) for details

---
## <a name="noNLlE98CP" id="noNLlE98CP">objArrayHandle</a>

### 概要(Summary)
objArrayOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_HANDLE(objArray         , is_objArray         )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific Handles for different oop types
    #define DEF_HANDLE(type, is_a)                   \
      class type##Handle;                            \
      class type##Handle: public Handle {            \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classobjArrayHandle.html) for details

---
## <a name="no2FjRWV1z" id="no2FjRWV1z">typeArrayHandle</a>

### 概要(Summary)
typeArrayOopDesc 用の Handle クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_HANDLE(typeArray        , is_typeArray        )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific Handles for different oop types
    #define DEF_HANDLE(type, is_a)                   \
      class type##Handle;                            \
      class type##Handle: public Handle {            \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classtypeArrayHandle.html) for details

---
## <a name="no51I7dDH4" id="no51I7dDH4">instanceKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が instanceKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(instanceKlass         , oop_is_instance_slow )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classinstanceKlassHandle.html) for details

---
## <a name="noJRUBjyHh" id="noJRUBjyHh">methodKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が methodKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(methodKlass           , oop_is_method        )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classmethodKlassHandle.html) for details

---
## <a name="noQKBjOc8g" id="noQKBjOc8g">constMethodKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が constMethodKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(constMethodKlass      , oop_is_constMethod   )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classconstMethodKlassHandle.html) for details

---
## <a name="noOeEP91Dp" id="noOeEP91Dp">klassKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が klassKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(klassKlass            , oop_is_klass         )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classklassKlassHandle.html) for details

---
## <a name="no9XLAfRXA" id="no9XLAfRXA">arrayKlassKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が arrayKlassKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(arrayKlassKlass       , oop_is_arrayKlass    )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classarrayKlassKlassHandle.html) for details

---
## <a name="nohfbH-q3Q" id="nohfbH-q3Q">objArrayKlassKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が objArrayKlassKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(objArrayKlassKlass    , oop_is_objArrayKlass )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classobjArrayKlassKlassHandle.html) for details

---
## <a name="noLcM0vxpc" id="noLcM0vxpc">typeArrayKlassKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が typeArrayKlassKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(typeArrayKlassKlass   , oop_is_typeArrayKlass)
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classtypeArrayKlassKlassHandle.html) for details

---
## <a name="noiw8Uzf-t" id="noiw8Uzf-t">arrayKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が arrayKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(arrayKlass            , oop_is_array         )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classarrayKlassHandle.html) for details

---
## <a name="no3Ri2OaVj" id="no3Ri2OaVj">typeArrayKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が typeArrayKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(typeArrayKlass        , oop_is_typeArray_slow)
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classtypeArrayKlassHandle.html) for details

---
## <a name="noY_W0TNCR" id="noY_W0TNCR">objArrayKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が objArrayKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(objArrayKlass         , oop_is_objArray_slow )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classobjArrayKlassHandle.html) for details

---
## <a name="no45oLBOrA" id="no45oLBOrA">constantPoolKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が constantPoolKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(constantPoolKlass     , oop_is_constantPool  )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).



### 詳細(Details)
See: [here](../doxygen/classconstantPoolKlassHandle.html) for details

---
## <a name="noRb2qu9rU" id="noRb2qu9rU">constantPoolCacheKlassHandle</a>

### 概要(Summary)
KlassHandle クラスのサブクラス.
このクラスは Klass 種別が constantPoolCacheKlass の場合用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    DEF_KLASS_HANDLE(constantPoolCacheKlass, oop_is_constantPool  )
```

なお, このクラスはソースコード上で直接定義はされておらず, 以下のマクロを使って間接的に定義されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Specific KlassHandles for different Klass types
    
    #define DEF_KLASS_HANDLE(type, is_a)             \
      class type##Handle : public KlassHandle {      \
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classconstantPoolCacheKlassHandle.html) for details

---
## <a name="no8KqXvHt4" id="no8KqXvHt4">HandleArea</a>

### 概要(Summary)
Handle クラス用の補助クラス.

Handle が必要とするデータを確保するための Arena クラス

(なお, Arena クラスなのでこの領域はスレッドローカル).


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // Thread local handle area
    
    class HandleArea: public Arena {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Thread オブジェクトの _handle_area フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
    class Thread: public ThreadShadow {
    ...
     protected:
    ...
      // Thread local handle area for allocation of handles within the VM
      HandleArea* _handle_area;
```

#### 生成箇所(where its instances are created)
Thread::Thread() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classHandleArea.html) for details

---
## <a name="no7QqAmO2A" id="no7QqAmO2A">HandleMark</a>

### 概要(Summary)
HandleArena クラス用のユーティリティ・クラス.

Handle の確保/開放をソースコード上のスコープに合わせて自動で行うことができる.
(参考: [Region-based memory management](http://en.wikipedia.org/wiki/Region-based_memory_management)).

コンストラクタではその時点での HandleArea の先頭位置(top 位置)を記録し, 
デストラクタで記録した位置まで戻す (= その時点までに確保したハンドル領域を解放する).


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    //------------------------------------------------------------------------------------------------------------------------
    // Handles are allocated in a (growable) thread local handle area. Deallocation
    // is managed using a HandleMark. It should normally not be necessary to use
    // HandleMarks manually.
    //
    // A HandleMark constructor will record the current handle area top, and the
    // desctructor will reset the top, destroying all handles allocated in between.
    // The following code will therefore NOT work:
    //
    //   Handle h;
    //   {
    //     HandleMark hm;
    //     h = Handle(obj);
    //   }
    //   h()->print();       // WRONG, h destroyed by HandleMark destructor.
    //
    // If h has to be preserved, it can be converted to an oop or a local JNI handle
    // across the HandleMark boundary.
    
    // The base class of HandleMark should have been StackObj but we also heap allocate
    // a HandleMark when a thread is created.
    
    class HandleMark {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classHandleMark.html) for details

---
## <a name="noetnh7N_B" id="noetnh7N_B">NoHandleMark</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時以外には空のクラスとして定義される).

「あるコード範囲で Handle の確保操作が起きない」ということをコード上に明示したい場合に使用される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    // A NoHandleMark stack object will verify that no handles are allocated
    // in its scope. Enabled in debug mode only.
    
    class NoHandleMark: public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で NoHandleMark 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
`#ifdef ASSERT` でなければ, 中身のないクラスとして定義される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    #ifdef ASSERT
      NoHandleMark();
      ~NoHandleMark();
    #else
      NoHandleMark()  {}
      ~NoHandleMark() {}
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classNoHandleMark.html) for details

---
## <a name="noC_9tJZH_" id="noC_9tJZH_">ResetNoHandleMark</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時以外には空のクラスとして定義される).

NoHandleMark の働きを一時的に無効化したい場合に使用する
(ResetNoHandleMark のコンストラクタが呼ばれてからデストラクタが呼ばれるまでの間は NoHandleMark の効果がキャンセルされる).


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    class ResetNoHandleMark: public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で ResetNoHandleMark 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
`#ifdef ASSERT` でなければ, 中身のないクラスとして定義される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/handles.hpp))
    #ifdef ASSERT
      ResetNoHandleMark();
      ~ResetNoHandleMark();
    #else
      ResetNoHandleMark()  {}
      ~ResetNoHandleMark() {}
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classResetNoHandleMark.html) for details

---
