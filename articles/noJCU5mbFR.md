---
layout: default
title: instanceOopDesc クラス 
---
[Top](../index.html)

#### instanceOopDesc クラス 



---
## <a name="noeRe4yZr4" id="noeRe4yZr4">instanceOopDesc</a>

### 概要(Summary)
Java レベルでのインスタンスオブジェクトを表すためのクラス.
1つの instanceOopDesc オブジェクトが JVM 上での 1つのインスタンスオブジェクトに対応する.


```cpp
    ((cite: hotspot/src/share/vm/oops/instanceOop.hpp))
    // An instanceOop is an instance of a Java Class
    // Evaluating "new HashTable()" will create an instanceOop.
    
    class instanceOopDesc : public oopDesc {
```

### 内部構造(Internal structure)
スーパークラスである oopDesc クラスの mark フィールド/klass フィールドの後ろに各インスタンス毎のデータを格納している模様.

        markOop
        klassOop
        data      // 各インスタンスごとのデータを格納

### 備考(Notes)
なお, 実際の使用箇所では instanceOop という別名(もしくはラッパークラス)で使われることが多い (See: instanceOop).




### 詳細(Details)
See: [here](../doxygen/classinstanceOopDesc.html) for details

---
