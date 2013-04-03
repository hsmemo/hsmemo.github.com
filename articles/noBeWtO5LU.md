---
layout: default
title: ciKlass クラス 
---
[Top](../index.html)

#### ciKlass クラス 



---
## <a name="noz-joVxSG" id="noz-joVxSG">ciKlass</a>

### 概要(Summary)
ciType クラスのサブクラスの1つ. klassOopDesc 用の ciObject クラス.

なお, klassOopDesc では実際のクラスの違いは埋め込まれた Klass オブジェクトの違いとして表現されているが, 
ciKlass では Klass 種別に対応したサブクラスが作られている
(例えば, objArrayKlass が埋め込まれた klassOop は ciObjArrayKlass というクラスで表現される, 等).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/ci/ciKlass.hpp))
    // ciKlass
    //
    // This class and its subclasses represent klassOops in the
    // HotSpot virtual machine.  In the vm, each klassOop contains an
    // embedded Klass object.  ciKlass is subclassed to explicitly
    // represent the kind of Klass embedded in the klassOop.  For
    // example, a klassOop with an embedded objArrayKlass object is
    // represented in the ciObject hierarchy by the class
    // ciObjArrayKlass.
    class ciKlass : public ciType {
```




### 詳細(Details)
See: [here](../doxygen/classciKlass.html) for details

---
