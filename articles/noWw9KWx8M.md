---
layout: default
title: ClassFileStream クラス 
---
[Top](../index.html)

#### ClassFileStream クラス 



---
## <a name="noYmmXBwtc" id="noYmmXBwtc">ClassFileStream</a>

### 概要(Summary)
ClassLoader クラス用の補助クラス (See: [here](no7882m2Z.html) and [here](no7882ALm.html) for details).

クラスファイルの読み込み処理で使用される一時オブジェクト(ResourceObjクラス).
クラスファイルに適した読み込み用のメソッド (get_u2(), get_u4(), etc) が定義されている.

なおコメントによると, 使う人がバッファの開放に責任を持たなければいけない, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/classfile/classFileStream.hpp))
    // Input stream for reading .class file
    //
    // The entire input stream is present in a buffer allocated by the caller.
    // The caller is responsible for deallocating the buffer and for using
    // ResourceMarks appropriately when constructing streams.
    
    class ClassFileStream: public ResourceObj {
```

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ClassPathDirEntry::open_stream()

* ClassPathZipEntry::open_stream()

* ClassFileParser::parseClassFile()
  
  (これは javaagent によってクラスファイルが変更された場合用)




### 詳細(Details)
See: [here](../doxygen/classClassFileStream.html) for details

---
