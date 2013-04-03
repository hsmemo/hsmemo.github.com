---
layout: default
title: ciCallProfile クラス 
---
[Top](../index.html)

#### ciCallProfile クラス 



---
## <a name="nol5XU2AxP" id="nol5XU2AxP">ciCallProfile</a>

### 概要(Summary)
JIT Compiler から各メソッド呼び出し箇所 (invoke*バイトコード) のプロファイル情報にアクセスするための一時オブジェクト(StackObjクラス).
1つの ciCallProfile オブジェクトが 1つのメソッド呼び出し箇所に対応する.


```
    ((cite: hotspot/src/share/vm/ci/ciCallProfile.hpp))
    // ciCallProfile
    //
    // This class is used to determine the frequently called method
    // at some call site
    class ciCallProfile : StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. ciMethod::call_profile_at_bci() で, 指定したバイトコードに対応する ciCallProfile オブジェクトを生成する.
2. ciCallProfile クラスの各メソッドで情報を取得できる.

#### 使用箇所(where its instances are used)
ciMethod::call_profile_at_bci() は以下の箇所で(のみ)呼び出されている.

```
Compile::call_generator()
-> ciMethod::call_profile_at_bci()

GraphKit::maybe_cast_profiled_receiver()
-> ciMethod::call_profile_at_bci()
```

### 内部構造(Internal structure)
実際のプロファイル情報は, 
対応箇所の CounterData や ReceiverTypeData から取得されている
(See: ciMethod::call_profile_at_bci()).




### 詳細(Details)
See: [here](../doxygen/classciCallProfile.html) for details

---
