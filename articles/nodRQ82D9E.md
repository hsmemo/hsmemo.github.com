---
layout: default
title: IdealKit クラス関連のクラス (IdealVariable, IdealKit)
---
[Top](../index.html)

#### IdealKit クラス関連のクラス (IdealVariable, IdealKit)

これらは, C2 JIT Compiler 用のクラス.
より具体的に言うと, 高レベル中間語の構築作業を容易化するための補助クラス.


### クラス一覧(class list)

  * [IdealKit](#noVWUbfe3d)
  * [IdealVariable](#noGkw2yMFj)


---
## <a name="noVWUbfe3d" id="noVWUbfe3d">IdealKit</a>

### 概要(Summary)
GraphKit クラスやそのサブクラス (より具体的に言うと GraphKit クラスと LibraryCallKit クラス) 内で使用される補助クラス.

Ideal を構築する作業用の一時オブジェクト(StackObjクラス).
複雑な Ideal グラフ (if や loop 等を含むもの) を作りたい場合に,
その構築を簡単にするメソッドを提供している.


```cpp
    ((cite: hotspot/src/share/vm/opto/idealKit.hpp))
    class IdealKit: public StackObj {
```


```cpp
    ((cite: hotspot/src/share/vm/opto/idealKit.hpp))
    //-----------------------------------------------------------------------------
    //----------------------------IdealKit-----------------------------------------
    // Set of utilities for creating control flow and scalar SSA data flow.
    // Control:
    //    if_then(left, relop, right)
    //    else_ (optional)
    //    end_if
    //    loop(iv variable, initial, relop, limit)
    //       - sets iv to initial for first trip
    //       - exits when relation on limit is true
    //       - the values of initial and limit should be loop invariant
    //       - no increment, must be explicitly coded
    //       - final value of iv is available after end_loop (until dead())
    //    end_loop
    //    make_label(number of gotos)
    //    goto_(label)
    //    bind(label)
    // Data:
    //    ConI(integer constant)     - create an integer constant
    //    set(variable, value)       - assignment
    //    value(variable)            - reference value
    //    dead(variable)             - variable's value is no longer live
    //    increment(variable, value) - increment variable by value
    //    simple operations: AddI, SubI, AndI, LShiftI, etc.
    // Example:
    //    Node* limit = ??
    //    IdealVariable i(kit), j(kit);
    //    declarations_done();
    //    Node* exit = make_label(1); // 1 goto
    //    set(j, ConI(0));
    //    loop(i, ConI(0), BoolTest::lt, limit); {
    //       if_then(value(i), BoolTest::gt, ConI(5)) {
    //         set(j, ConI(1));
    //         goto_(exit); dead(i);
    //       } end_if();
    //       increment(i, ConI(1));
    //    } end_loop(); dead(i);
    //    bind(exit);
    //
    // See string_indexOf for a more complete example.
```

### 使われ方(Usage)
#### 使用例(usage examples)
(使用時には IdealKit のコンストラクタに GraphKit オブジェクトを渡す.
 IdealKit のメソッドを呼び出すと内部で GraphKit のメソッドが呼び出される)

以下のように "__ loop()" や "__ if_then()" と書くだけで Ideal が構築できる.


```cpp
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
      IdealKit kit(this, false, true);
    #define __ kit.
      Node* zero             = __ ConI(0);
      Node* one              = __ ConI(1);
      Node* cache            = __ ConI(cache_i);
      Node* md2              = __ ConI(md2_i);
      Node* lastChar         = __ ConI(target_array->char_at(target_length - 1));
      Node* targetCount      = __ ConI(target_length);
      Node* targetCountLess1 = __ ConI(target_length - 1);
      Node* targetOffset     = __ ConI(targetOffset_i);
      Node* sourceEnd        = __ SubI(__ AddI(sourceOffset, sourceCount), targetCountLess1);
    
      IdealVariable rtn(kit), i(kit), j(kit); __ declarations_done();
      Node* outer_loop = __ make_label(2 /* goto */);
      Node* return_    = __ make_label(1);
    
      __ set(rtn,__ ConI(-1));
      __ loop(this, nargs, i, sourceOffset, BoolTest::lt, sourceEnd); {
           Node* i2  = __ AddI(__ value(i), targetCountLess1);
           // pin to prohibit loading of "next iteration" value which may SEGV (rare)
           Node* src = load_array_element(__ ctrl(), source, i2, TypeAryPtr::CHARS);
           __ if_then(src, BoolTest::eq, lastChar, unlikely); {
             __ loop(this, nargs, j, zero, BoolTest::lt, targetCountLess1); {
                  Node* tpj = __ AddI(targetOffset, __ value(j));
                  Node* targ = load_array_element(no_ctrl, target, tpj, target_type);
                  Node* ipj  = __ AddI(__ value(i), __ value(j));
                  Node* src2 = load_array_element(no_ctrl, source, ipj, TypeAryPtr::CHARS);
                  __ if_then(targ, BoolTest::ne, src2); {
                    __ if_then(__ AndI(cache, __ LShiftI(one, src2)), BoolTest::eq, zero); {
                      __ if_then(md2, BoolTest::lt, __ AddI(__ value(j), one)); {
                        __ increment(i, __ AddI(__ value(j), one));
                        __ goto_(outer_loop);
                      } __ end_if(); __ dead(j);
                    }__ end_if(); __ dead(j);
                    __ increment(i, md2);
                    __ goto_(outer_loop);
                  }__ end_if();
                  __ increment(j, one);
             }__ end_loop(); __ dead(j);
             __ set(rtn, __ SubI(__ value(i), sourceOffset)); __ dead(i);
             __ goto_(return_);
           }__ end_if();
           __ if_then(__ AndI(cache, __ LShiftI(one, src)), BoolTest::eq, zero, likely); {
             __ increment(i, targetCountLess1);
           }__ end_if();
           __ increment(i, one);
           __ bind(outer_loop);
      }__ end_loop(); __ dead(i);
      __ bind(return_);
```

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* GraphKit::write_barrier_post()
* GraphKit::g1_write_barrier_pre()
* GraphKit::g1_write_barrier_post()
* LibraryCallKit::string_indexOf()
* LibraryCallKit::insert_g1_pre_barrier()
* LibraryCallKit::inline_unsafe_access()




### 詳細(Details)
See: [here](../doxygen/classIdealKit.html) for details

---
## <a name="noGkw2yMFj" id="noGkw2yMFj">IdealVariable</a>

### 概要(Summary)
IdealKit クラス用の補助クラス(StackObjクラス).

IdealKit クラスによる Ideal 構築時に「変数」として利用できる補助クラス.


```cpp
    ((cite: hotspot/src/share/vm/opto/idealKit.hpp))
    // Variable definition for IdealKit
    class IdealVariable: public StackObj {
```

### 使われ方(Usage)
#### 使用例(usage examples)
IdealKit::set()/IdealKit::value() で値を設定／参照できる他, 
IdealKit::loop() の制御変数等としても利用できる.


```cpp
    ((cite: hotspot/src/share/vm/opto/library_call.cpp))
      IdealKit kit(this, false, true);
    #define __ kit.
      Node* zero             = __ ConI(0);
      Node* one              = __ ConI(1);
      Node* cache            = __ ConI(cache_i);
      Node* md2              = __ ConI(md2_i);
      Node* lastChar         = __ ConI(target_array->char_at(target_length - 1));
      Node* targetCount      = __ ConI(target_length);
      Node* targetCountLess1 = __ ConI(target_length - 1);
      Node* targetOffset     = __ ConI(targetOffset_i);
      Node* sourceEnd        = __ SubI(__ AddI(sourceOffset, sourceCount), targetCountLess1);
    
      IdealVariable rtn(kit), i(kit), j(kit); __ declarations_done();
      Node* outer_loop = __ make_label(2 /* goto */);
      Node* return_    = __ make_label(1);
    
      __ set(rtn,__ ConI(-1));
      __ loop(this, nargs, i, sourceOffset, BoolTest::lt, sourceEnd); {
           Node* i2  = __ AddI(__ value(i), targetCountLess1);
           // pin to prohibit loading of "next iteration" value which may SEGV (rare)
           Node* src = load_array_element(__ ctrl(), source, i2, TypeAryPtr::CHARS);
           __ if_then(src, BoolTest::eq, lastChar, unlikely); {
             __ loop(this, nargs, j, zero, BoolTest::lt, targetCountLess1); {
                  Node* tpj = __ AddI(targetOffset, __ value(j));
                  Node* targ = load_array_element(no_ctrl, target, tpj, target_type);
                  Node* ipj  = __ AddI(__ value(i), __ value(j));
                  Node* src2 = load_array_element(no_ctrl, source, ipj, TypeAryPtr::CHARS);
                  __ if_then(targ, BoolTest::ne, src2); {
                    __ if_then(__ AndI(cache, __ LShiftI(one, src2)), BoolTest::eq, zero); {
                      __ if_then(md2, BoolTest::lt, __ AddI(__ value(j), one)); {
                        __ increment(i, __ AddI(__ value(j), one));
                        __ goto_(outer_loop);
                      } __ end_if(); __ dead(j);
                    }__ end_if(); __ dead(j);
                    __ increment(i, md2);
                    __ goto_(outer_loop);
                  }__ end_if();
                  __ increment(j, one);
             }__ end_loop(); __ dead(j);
             __ set(rtn, __ SubI(__ value(i), sourceOffset)); __ dead(i);
             __ goto_(return_);
           }__ end_if();
           __ if_then(__ AndI(cache, __ LShiftI(one, src)), BoolTest::eq, zero, likely); {
             __ increment(i, targetCountLess1);
           }__ end_if();
           __ increment(i, one);
           __ bind(outer_loop);
      }__ end_loop(); __ dead(i);
      __ bind(return_);
```

#### 使用箇所(where its instances are used)
LibraryCallKit::string_indexOf() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classIdealVariable.html) for details

---
