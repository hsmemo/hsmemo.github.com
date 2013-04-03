---
layout: default
title: Klass クラス関連のクラス (Klass_vtbl, Klass)
---
[Top](../index.html)

#### Klass クラス関連のクラス (Klass_vtbl, Klass)



### クラス一覧(class list)

  * [Klass_vtbl](#noVhOUHTRt)
  * [Klass](#no1wOZOPAn)


---
## <a name="noVhOUHTRt" id="noVhOUHTRt">Klass_vtbl</a>

### 概要(Summary)
全ての Klass クラスの基底クラス.

Klass 用の C++ の vtable を保持するためのクラス.
Klass オブジェクト中での vtable の位置を制限したいためこの基底クラスが存在する, とのこと.


```
    ((cite: hotspot/src/share/vm/oops/klass.hpp))
    // Holder (or cage) for the C++ vtable of each kind of Klass.
    // We want to tightly constrain the location of the C++ vtable in the overall layout.
    class Klass_vtbl {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

### 内部構造(Internal structure)
内部では, vtable の位置を確定させるため, 空の virtual メソッドが定義されている.


```
    ((cite: hotspot/src/share/vm/oops/klass.hpp))
      // The following virtual exists only to force creation of a C++ vtable,
      // so that this class truly is the location of the vtable of all Klasses.
      virtual void unused_initial_virtual() { }
```

また, Klass オブジェクトを生成するためのファクトリメソッドも用意されている.


```
    ((cite: hotspot/src/share/vm/oops/klass.hpp))
      // The following virtual makes Klass_vtbl play a second role as a
      // factory protocol for subclasses of Klass ("sub-Klasses").
      // Here's how it works....
      //
      // This VM uses metaobjects as factories for their instances.
      //
      // In order to initialize the C++ vtable of a new instance, its
      // metaobject is forced to use the C++ placed new operator to
      // allocate the instance.  In a typical C++-based system, each
      // sub-class would have its own factory routine which
      // directly uses the placed new operator on the desired class,
      // and then calls the appropriate chain of C++ constructors.
      //
      // However, this system uses shared code to performs the first
      // allocation and initialization steps for all sub-Klasses.
      // (See base_create_klass() and base_create_array_klass().)
      // This does not factor neatly into a hierarchy of C++ constructors.
      // Each caller of these shared "base_create" routines knows
      // exactly which sub-Klass it is creating, but the shared routine
      // does not, even though it must perform the actual allocation.
      //
      // Therefore, the caller of the shared "base_create" must wrap
      // the specific placed new call in a virtual function which
      // performs the actual allocation and vtable set-up.  That
      // virtual function is here, Klass_vtbl::allocate_permanent.
      //
      // The arguments to Universe::allocate_permanent() are passed
      // straight through the placed new operator, which in turn
      // obtains them directly from this virtual call.
      //
      // This virtual is called on a temporary "example instance" of the
      // sub-Klass being instantiated, a C++ auto variable.  The "real"
      // instance created by this virtual is on the VM heap, where it is
      // equipped with a klassOopDesc header.
      //
      // It is merely an accident of implementation that we use "example
      // instances", but that is why the virtual function which implements
      // each sub-Klass factory happens to be defined by the same sub-Klass
      // for which it creates instances.
      //
      // The vtbl_value() call (see below) is used to strip away the
      // accidental Klass-ness from an "example instance" and present it as
      // a factory.  Think of each factory object as a mere container of the
      // C++ vtable for the desired sub-Klass.  Since C++ does not allow
      // direct references to vtables, the factory must also be delegated
      // the task of allocating the instance, but the essential point is
      // that the factory knows how to initialize the C++ vtable with the
      // right pointer value.  All other common initializations are handled
      // by the shared "base_create" subroutines.
      //
      virtual void* allocate_permanent(KlassHandle& klass, int size, TRAPS) const = 0;
      void post_new_init_klass(KlassHandle& klass, klassOop obj, int size) const;
    
      // Every subclass on which vtbl_value is called must include this macro.
      // Delay the installation of the klassKlass pointer until after the
      // the vtable for a new klass has been installed (after the call to new()).
    #define DEFINE_ALLOCATE_PERMANENT(thisKlass)                                  \
      void* allocate_permanent(KlassHandle& klass_klass, int size, TRAPS) const { \
        void* result = new(klass_klass, size, THREAD) thisKlass();                \
        if (HAS_PENDING_EXCEPTION) return NULL;                                   \
        klassOop new_klass = ((Klass*) result)->as_klassOop();                    \
        OrderAccess::storestore();                                                \
        post_new_init_klass(klass_klass, new_klass, size);                        \
        return result;                                                            \
      }
```




### 詳細(Details)
See: [here](../doxygen/classKlass__vtbl.html) for details

---
## <a name="no1wOZOPAn" id="no1wOZOPAn">Klass</a>

### 概要(Summary)
(事実上)全ての Klass クラスの基底クラス
(正確に言うと全ての基底クラスは Klass_vtbl だが, Klass_vtbl にはあまり中身がないので実質的な基底クラスは Klass).


```
    ((cite: hotspot/src/share/vm/oops/klass.hpp))
    // A Klass is the part of the klassOop that provides:
    //  1: language level class object (method dictionary etc.)
    //  2: provide vm dispatch behavior for the object
    // Both functions are combined into one C++ class. The toplevel class "Klass"
    // implements purpose 1 whereas all subclasses provide extra virtual functions
    // for purpose 2.
```


```
    ((cite: hotspot/src/share/vm/oops/klass.hpp))
    class Klass : public Klass_vtbl {
```

なお, oop と Klass を分けているのは oop 内に vtable を持たせたくないため, とのこと.
そのため, 現在は oop は virtual method を持たず, 必要な場合は Klass の方に持たせている.


```
    ((cite: hotspot/src/share/vm/oops/klass.hpp))
    // One reason for the oop/klass dichotomy in the implementation is
    // that we don't want a C++ vtbl pointer in every object.  Thus,
    // normal oops don't have any virtual functions.  Instead, they
    // forward all "virtual" functions to their klass, which does have
    // a vtbl and does the C++ dispatch depending on the object's
    // actual type.  (See oop.inline.hpp for some of the forwarding code.)
    // ALL FUNCTIONS IMPLEMENTING THIS DISPATCH ARE PREFIXED WITH "oop_"!
```

### 内部構造(Internal structure)
メモリ上には klassOopDesc とセットで確保される (Klass は常に klassOopDesc の一部として存在する) (See: KlassOopDesc).

メモリ上でのレイアウトは以下の通り.


```
    ((cite: hotspot/src/share/vm/oops/klass.hpp))
    //  Klass layout:
    //    [header        ] klassOop
    //    [klass pointer ] klassOop
    //    [C++ vtbl ptr  ] (contained in Klass_vtbl)
    //    [layout_helper ]
    //    [super_check_offset   ] for fast subtype checks
    //    [secondary_super_cache] for fast subtype checks
    //    [secondary_supers     ] array of 2ndary supertypes
    //    [primary_supers 0]
    //    [primary_supers 1]
    //    [primary_supers 2]
    //    ...
    //    [primary_supers 7]
    //    [java_mirror   ]
    //    [super         ]
    //    [name          ]
    //    [first subklass]
    //    [next_sibling  ] link to chain additional subklasses
    //    [modifier_flags]
    //    [access_flags  ]
    //    [verify_count  ] - not in product
    //    [alloc_count   ]
    //    [last_biased_lock_bulk_revocation_time] (64 bits)
    //    [prototype_header]
    //    [biased_lock_revocation_count]
```

### 備考(Notes)
このクラスのサブクラスは以下のような継承関係を持つ.

(oopDesc の hierarchy にほぼ対応しているが, コメントにも書いてあるように完全に1対1ではない)

         Klass                     -- 全ての Klass の基底. abstract class
           instanceKlass           -- instanceOopDesc の Klass. いわゆる「Java のクラス」を表す
             instanceMirrorKlass   -- java.lang.Class (のサブクラス) 用の instanceOopDesc (特別な配慮が必要なので instanceKlass とは別に用意している)
             instanceRefKlass      -- java.lang.ref.Reference (のサブクラス) 用の instanceOopDesc (GC 時に特別な配慮が必要なので別に用意している)
           methodKlass             -- methodOopDesc の Klass
           constMethodKlass        -- constMethodOopDesc の Klass
           methodDataKlass         -- methodDataOopDesc の Klass
           klassKlass              -- klassOopDesc の Klass
             instanceKlassKlass    -- Klass として instanceKlass を持つ場合の klassOopDesc の Klass
             arrayKlassKlass       -- Klass として arrayKlass を持つ場合の klassOopDesc の Klass
               objArrayKlassKlass  -- Klass として objArrayKlass を持つ場合の klassOopDesc の Klass
               typeArrayKlassKlass -- Klass として typeArrayKlass を持つ場合の klassOopDesc の Klass
           arrayKlass              -- arrayOopDesc の Klass. いわゆる「Java の配列クラス」を表す. abstract class
             objArrayKlass         -- objArrayOopDesc の Klass. オブジェクトの配列に対する配列クラスを表す
             typeArrayKlass        -- typeArrayOopDesc の Klass. primitive 型の配列に対する配列クラスを表す
           constantPoolKlass       -- constantPoolOopDesc の Klass
           constantPoolCacheKlass  -- constantPoolCacheOopDesc の Klass
           compiledICHolderKlass   -- compiledICHolderOop の Klass




### 詳細(Details)
See: [here](../doxygen/classKlass.html) for details

---
