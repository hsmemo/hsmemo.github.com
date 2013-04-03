---
layout: default
title: ciExceptionHandler クラス 
---
[Top](../index.html)

#### ciExceptionHandler クラス 



---
## <a name="nozhOu8s3K" id="nozhOu8s3K">ciExceptionHandler</a>

### 概要(Summary)
JIT Compiler から例外ハンドラ情報にアクセスするための一時オブジェクト(ResourceObjクラス).
1つの ciExceptionHandler オブジェクトが 1つの例外ハンドラに対応する.


```
    ((cite: hotspot/src/share/vm/ci/ciExceptionHandler.hpp))
    // ciExceptionHandler
    //
    // This class represents an exception handler for a method.
    class ciExceptionHandler : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の箇所).

* 各 ciMethod オブジェクトの _exception_handlers フィールド
  
  そのメソッドの例外ハンドラ全てを格納したフィールド.
  
* 各 XHandler オブジェクトの _desc フィールド  (C1 の場合)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciMethod::load_code()
  
  (ciMethod::_exception_handlers フィールドの初期化用)

* GraphBuilder::inline_sync_entry()  (C1 の場合)




### 詳細(Details)
See: [here](../doxygen/classciExceptionHandler.html) for details

---
