---
layout: default
title: Class のロード/リンク/初期化(Loading/Linking/Initializing) 
---
[Up](no38NSe1ks.html) [Top](../index.html)

#### Class のロード/リンク/初期化(Loading/Linking/Initializing) 

--- 
## 概要(Summary)
(#Under Construction)

  * クラスのロード処理 (See: [here](nohUsh5oi7.html) for details)
  * クラスのリンク処理 (See: [here](noX5hsnWQw.html) for details)
  * クラスの初期化処理 (See: [here](no6dqMzJWt.html) for details)

## 参考(for your information)
* Java仮想マシン仕様 Chapter.5 "Loading, Linking, and Initializing"

## 備考(Notes)
各クラスの初期化状態は,
各 instanceKlass オブジェクトの _init_state フィールド (ClassState 型) で管理されている


```cpp
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
      ClassState      _init_state;           // state of class
```


```cpp
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
      // See "The Java Virtual Machine Specification" section 2.16.2-5 for a detailed description
      // of the class loading & initialization procedure, and the use of the states.
      enum ClassState {
        unparsable_by_gc = 0,               // object is not yet parsable by gc. Value of _init_state at object allocation.
        allocated,                          // allocated (but not yet linked)
        loaded,                             // loaded and inserted in class hierarchy (but not linked yet)
        linked,                             // successfully linked/verified (but not initialized yet)
        being_initialized,                  // currently running class initializer
        fully_initialized,                  // initialized (successfull final state)
        initialization_error                // error happened during initialization
      };
```




## Subcategories
* [Class のロード/リンク/初期化 ： ロード処理(Loading)](nohUsh5oi7.html)
* [Class のロード/リンク/初期化 ： リンク処理(Linking)](noX5hsnWQw.html)
* [Class のロード/リンク/初期化 ： 初期化処理(Initializing)](no6dqMzJWt.html)



