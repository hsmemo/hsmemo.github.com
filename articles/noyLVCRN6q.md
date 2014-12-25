---
layout: default
title: PrivilegedElement クラス 
---
[Top](../index.html)

#### PrivilegedElement クラス 



---
## <a name="nofobfTffM" id="nofobfTffM">PrivilegedElement</a>

### 概要(Summary)
Java のセキュリティ機能 (より具体的に言うと java.security.AccessController.doPrivileged() の機能) を実現するためのクラス
(See: [here](no3718WD2.html) for details).

doPrivileged() されたという情報を記録しておくためのクラス. 
1つの PrivilegedElement オブジェクトが 1回の java.security.AccessController.doPrivileged() に対応する.


```cpp
    ((cite: hotspot/src/share/vm/prims/privilegedStack.hpp))
    class PrivilegedElement VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 JavaThread オブジェクトの _privileged_stack_top フィールドに(のみ)格納されている.

(正確には, このフィールドは PrivilegedElement の線形リストを格納するフィールド.
PrivilegedElement オブジェクトは _next フィールドで次の PrivilegedElement オブジェクトを指せる構造になっている.
その JavaThread オブジェクト内で生成した PrivilegedElement オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 生成箇所(where its instances are created)
(ValueObj クラスなので「生成」というのは少し違和感があるが)
JVM_DoPrivileged() 内の局所変数として(のみ)生成される.




### 詳細(Details)
See: [here](../doxygen/classPrivilegedElement.html) for details

---
