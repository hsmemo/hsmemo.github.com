---
layout: default
title: MethodComparator クラスおよびその補助クラス (MethodComparator, BciMap)
---
[Top](../index.html)

#### MethodComparator クラスおよびその補助クラス (MethodComparator, BciMap)



### クラス一覧(class list)

  * [MethodComparator](#nohoV2Dry0)
  * [BciMap](#noDO85knvR)


---
## <a name="nohoV2Dry0" id="nohoV2Dry0">MethodComparator</a>

### 概要(Summary)
EMCP (Equivalent modulo Constant Pool) のための判定処理を納めた名前空間
(このクラスは AllStatic ではないが, static なフィールド／メソッドしか持たない).

(なお, EMCP については RedefineClass() 参照)

二つのメソッドが EMCP であるかどうかや "switchable"(??#TODO) であるかどうかを判定する機能を提供している.

(public method として, methods_EMCP() 以外に methods_switchable() というメソッドも持っているが, こちらは何をするメソッド? 
 どこからも使われてないのでよく分からん. #TODO)


```cpp
    ((cite: hotspot/src/share/vm/prims/methodComparator.hpp))
    // methodComparator provides an interface for determining if methods of
    // different versions of classes are equivalent or switchable
    
    class MethodComparator {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* VM_RedefineClasses::check_methods_and_mark_as_obsolete()  (See: [here](no2935-Vj.html) for details)
* Rewriter::relocate_and_link()
  (ただしこちらは #ifdef ASSERT かつ StressMethodComparator オプションが指定されている場合にのみ使用される)


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp))
    void VM_RedefineClasses::check_methods_and_mark_as_obsolete(
           BitMap *emcp_methods, int * emcp_method_count_p) {
    ...
        if (MethodComparator::methods_EMCP(old_method, new_method)) {
```


```cpp
    ((cite: hotspot/src/share/vm/interpreter/rewriter.cpp))
    void Rewriter::relocate_and_link(instanceKlassHandle this_oop,
                                     objArrayHandle methods, TRAPS) {
    ...
        // This is for JVMTI and unrelated to relocator but the last thing we do
    #ifdef ASSERT
        if (StressMethodComparator) {
    ...
            bool z = MethodComparator::methods_EMCP(m(),
                       (methodOop)methods->obj_at(j));
            if (j == i && !z) {
              tty->print("MethodComparator FAIL: "); m->print(); m->print_codes();
              assert(z, "method must compare equal to itself");
```

### 内部構造(Internal structure)
定義されている public メソッドは, 以下の2つだけ.

(しかも, MethodComparator::methods_switchable() の方はどこからも使用されていない)

```cpp
    ((cite: hotspot/src/share/vm/prims/methodComparator.hpp))
      // Check if the new method is equivalent to the old one modulo constant pool (EMCP).
      // Intuitive definition: two versions of the same method are EMCP, if they don't differ
      // on the source code level. Practically, we check whether the only difference between
      // method versions is some constantpool indices embedded into the bytecodes, and whether
      // these indices eventually point to the same constants for both method versions.
      static bool methods_EMCP(methodOop old_method, methodOop new_method);
    
      static bool methods_switchable(methodOop old_method, methodOop new_method, BciMap &bci_map);
```




### 詳細(Details)
See: [here](../doxygen/classMethodComparator.html) for details

---
## <a name="noDO85knvR" id="noDO85knvR">BciMap</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらないような...)

MethodComparator::methods_switchable() に引数として渡され, その中の処理パスで(のみ)使用されるように見える.
MethodComparator::methods_switchable() 自体がよく分からないので詳細不明.


```cpp
    ((cite: hotspot/src/share/vm/prims/methodComparator.hpp))
    // ByteCode Index Map. For two versions of the same method, where the new version may contain
    // fragments not found in the old version, provides a mapping from an index of a bytecode in
    // the old method to the index of the same bytecode in the new method.
    
    class BciMap {
```

### 使われ方(Usage)
(インスタンスは MethodComparator クラスの _bci_map フィールドに格納できるようだが... #TODO)
(このフィールドは MethodComparator::methods_switchable() に渡された引数がセットされる模様.
このフィールドは MethodComparator::args_same() 内で参照されている.
ただし MethodComparator::methods_EMCP() から呼び出された場合は参照せず,
_switchable_test という MethodComparator::methods_switchable() から呼ばれたときしか
true にならない変数がセットされていないと使用されない模様)




### 詳細(Details)
See: [here](../doxygen/classBciMap.html) for details

---
