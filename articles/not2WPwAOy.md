---
layout: default
title: VM_RedefineClasses クラス (VM_RedefineClasses, 及びその補助クラス(TransferNativeFunctionRegistration))
---
[Top](../index.html)

#### VM_RedefineClasses クラス (VM_RedefineClasses, 及びその補助クラス(TransferNativeFunctionRegistration))



### クラス一覧(class list)

  * [VM_RedefineClasses](#noo1EhmGjg)
  * [TransferNativeFunctionRegistration](#noyeDTGuKD)


---
## <a name="noo1EhmGjg" id="noo1EhmGjg">VM_RedefineClasses</a>

### 概要(Summary)
JVMTI の RedefineClass 機能 (IsModifiableClass(), RedefineClasses(), RetransformClasses()) の実装のためのクラス
(See: [here](no2935xLd.html) and [here](no2935-Vj.html) for details).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiRedefineClasses.hpp))
    class VM_RedefineClasses: public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JvmtiEnv::RetransformClasses()
* JvmtiEnv::RedefineClasses()



### 詳細(Details)
See: [here](../doxygen/classVM__RedefineClasses.html) for details

---
## <a name="noyeDTGuKD" id="noyeDTGuKD">TransferNativeFunctionRegistration</a>

### 概要(Summary)
VM_RedefineClasses クラス内で使われる補助クラス.

RedefineClass 時に, 
古いクラスに native method としてバインドされているネイティブ関数を
(Redefine 後の)新しいクラスへ登録し直す処理を行う.

なお, SetNativeMethodPrefix() や SetNativeMethodPrefies() によって prefix が変わることもあるが, 
そういったケースについてもこのクラス内で処理している模様.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp))
    // This internal class transfers the native function registration from old methods
    // to new methods.  It is designed to handle both the simple case of unchanged
    // native methods and the complex cases of native method prefixes being added and/or
    // removed.
    // It expects only to be used during the VM_RedefineClasses op (a safepoint).
    //
    // This class is used after the new methods have been installed in "the_class".
    //
    // So, for example, the following must be handled.  Where 'm' is a method and
    // a number followed by an underscore is a prefix.
    //
    //                                      Old Name    New Name
    // Simple transfer to new method        m       ->  m
    // Add prefix                           m       ->  1_m
    // Remove prefix                        1_m     ->  m
    // Simultaneous add of prefixes         m       ->  3_2_1_m
    // Simultaneous removal of prefixes     3_2_1_m ->  m
    // Simultaneous add and remove          1_m     ->  2_m
    // Same, caused by prefix removal only  3_2_1_m ->  3_2_m
    //
    class TransferNativeFunctionRegistration {
```

### 使われ方(Usage)
VM_RedefineClasses::transfer_old_native_function_registrations() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている (See: [here](no2935-Vj.html) for details).

```
VM_RedefineClasses::doit()
-> VM_RedefineClasses::redefine_single_class()
   -> VM_RedefineClasses::transfer_old_native_function_registrations()
```




### 詳細(Details)
See: [here](../doxygen/classTransferNativeFunctionRegistration.html) for details

---
