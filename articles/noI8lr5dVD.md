---
layout: default
title: arrayKlass クラス 
---
[Top](../index.html)

#### arrayKlass クラス 



---
## <a name="no36CzxxzN" id="no36CzxxzN">arrayKlass</a>

### 概要(Summary)
arrayOopDesc 用の Klass クラスの基底クラス.


```
    ((cite: hotspot/src/share/vm/oops/arrayKlass.hpp))
    // arrayKlass is the abstract baseclass for all array classes
    
    class arrayKlass: public Klass {
```

なお, このクラス自体は abstract class であり, 実際に使われるのは以下のサブクラス.

* typeArrayKlass
* objArrayKlass




### 詳細(Details)
See: [here](../doxygen/classarrayKlass.html) for details

---
