---
layout: default
title: InterpreterRuntime クラスの補助クラス (InterpreterRuntime::SignatureHandlerGenerator, SlowSignatureHandler)
---
[Top](../index.html)

#### InterpreterRuntime クラスの補助クラス (InterpreterRuntime::SignatureHandlerGenerator, SlowSignatureHandler)

これらは, JNI の機能(より具体的に言うと, ネイティブメソッドの呼び出し機能)を実現するためのクラス.
引数をネイティブの ABI にしたがった形に変換する処理を行う (See: [here](no3059asZ.html) for details).


### クラス一覧(class list)

  * [InterpreterRuntime::SignatureHandlerGenerator](#noGk_E8Nuq)
  * [SlowSignatureHandler](#noyT9WACRd)


---
## <a name="noGk_E8Nuq" id="noGk_E8Nuq">InterpreterRuntime::SignatureHandlerGenerator</a>

### 概要(Summary)
InterpreterRuntime クラス内で使用される補助クラス.

JNI の機能(より具体的に言うと, ネイティブメソッドの呼び出し機能)を実現するためのクラス.
引数をネイティブの ABI にしたがった形に変換する処理を行う. 

なお, 似た役割のクラスに SlowSignatureHandler がある. こちらは引数が複雑なネイティブメソッド用.
(逆に InterpreterRuntime::SignatureHandlerGenerator は引数が比較的簡単な場合に使用され, 
それぞれのネイティブメソッドに特注のシグネチャハンドラ(引数をネイティブの ABI にしたがった形に変換する関数)を生成する処理を行う)
(See: [here](no3059asZ.html) for details).


```cpp
    ((cite: hotspot/src/cpu/x86/vm/interpreterRT_x86.hpp))
    // native method calls
    
    class SignatureHandlerGenerator: public NativeSignatureIterator {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
SignatureHandlerLibrary::add() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
SignatureHandlerLibrary::add() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classInterpreterRuntime_1_1SignatureHandlerGenerator.html) for details

---
## <a name="noyT9WACRd" id="noyT9WACRd">SlowSignatureHandler</a>

### 概要(Summary)
InterpreterRuntime クラス内で使用される補助クラス.

JNI の機能(より具体的に言うと, ネイティブメソッドの呼び出し機能)を実現するためのクラス.
引数をネイティブの ABI にしたがった形に変換する処理を行う (See: [here](no3059asZ.html) for details).

なお, 似た役割のクラスに InterpreterRuntime::SignatureHandlerGenerator がある. こちらは引数が簡単なネイティブメソッド用.
(逆に SlowSignatureHandler は引数が複雑な場合に使用される)
(See: [here](no3059asZ.html) for details).

(なお, 32bit か 64bit かによってクラス定義が別になっている.
さらに, 64bit 版は #ifdef _WIN64 かどうかで 2種類のクラス定義が用意されている)


```cpp
    ((cite: hotspot/src/cpu/x86/vm/interpreterRT_x86_32.cpp))
    class SlowSignatureHandler: public NativeSignatureIterator {
```


```cpp
    ((cite: hotspot/src/cpu/x86/vm/interpreterRT_x86_64.cpp))
    #ifdef _WIN64
    class SlowSignatureHandler
      : public NativeSignatureIterator {
```


```cpp
    ((cite: hotspot/src/cpu/x86/vm/interpreterRT_x86_64.cpp))
    #else
    class SlowSignatureHandler
      : public NativeSignatureIterator {
```

### 使われ方(Usage)
InterpreterRuntime::slow_signature_handler() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classSlowSignatureHandler.html) for details

---
