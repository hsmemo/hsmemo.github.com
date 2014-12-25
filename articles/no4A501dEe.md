---
layout: default
title: JavaCalls クラス関連のクラス (JavaCallWrapper, JavaCallArguments, JavaCalls, 及びそれらの補助クラス(SignatureChekker))
---
[Top](../index.html)

#### JavaCalls クラス関連のクラス (JavaCallWrapper, JavaCallArguments, JavaCalls, 及びそれらの補助クラス(SignatureChekker))

これらは, HotSpot のランタイムから Java のコードを呼び出すためのクラス (See: [here](no3059iJu.html) for details).


### クラス一覧(class list)

  * [JavaCalls](#noaVmvTBg3)
  * [JavaCallWrapper](#nopDjyrPwg)
  * [JavaCallArguments](#notMiqrSFZ)
  * [SignatureChekker](#noVMAU0HMM)


---
## <a name="noaVmvTBg3" id="noaVmvTBg3">JavaCalls</a>

### 概要(Summary)
HotSpot のランタイムから Java のコードを呼び出す処理 (= Java のメソッドの実行を開始する処理) を納めた名前空間(AllStatic クラス) (See: [here](no3059iJu.html) for details).

Java のメソッドの開始する際には基本的にこのクラスが使用される. 以下は使用箇所の例.

* Java プログラムの実行を開始する際 (= main() メソッドを実行する際)
* 新しいスレッドの実行を開始する際 (= java.lang.Thread.run() メソッドを実行する際)
* ランタイムの処理の途中で Java のメソッドを実行する必要が生じた際 (クラスローディング処理, など)
* etc


```cpp
    ((cite: hotspot/src/share/vm/runtime/javaCalls.hpp))
    // All calls to Java have to go via JavaCalls. Sets up the stack frame
    // and makes sure that the last_Java_frame pointers are chained correctly.
    //
    
    class JavaCalls: AllStatic {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classJavaCalls.html) for details

---
## <a name="nopDjyrPwg" id="nopDjyrPwg">JavaCallWrapper</a>

### 概要(Summary)
JavaCalls クラス内で使用される補助クラス(StackObjクラス) (See: [here](no3059iJu.html) for details).

JavaCalls による Java コード呼び出しの前準備と後始末を行う (呼び出しの直前に生成され, 呼び出しが終わった直後に破棄される).
具体的には以下のような処理を行う.

* 呼び出し前に JNI ローカル参照フレームを新しいものに交換する
  (古い JNI ローカル参照フレームは待避しておき, 呼び出し後に元に戻す).
* 呼び出し前に JavaFrameAnchor をリセットする (呼び出し後に元に戻す).
* スレッドの状態を _thread_in_vm から _thread_in_Java に変更する.
* 「呼び出し先のメソッド」や「レシーバー」のポインタが GC 時に調査対象になるようにする.


```cpp
    ((cite: hotspot/src/share/vm/runtime/javaCalls.hpp))
    // A JavaCallWrapper is constructed before each JavaCall and destructed after the call.
    // Its purpose is to allocate/deallocate a new handle block and to save/restore the last
    // Java fp/sp. A pointer to the JavaCallWrapper is stored on the stack.
    
    class JavaCallWrapper: StackObj {
```

### 使われ方(Usage)
JavaCalls::call_helper() 内で(のみ)使用されている.

### 備考(Notes)
JavaCallWrapper が指しているポインタは GC で調査する必要がある.

このため GC 時には, 
スタックフレーム内に埋まっている JavaCallWrapper を見つけて JavaCallWrapper::oops_do() を実行する, 
という処理が行われる
(See: frame::entry_frame_call_wrapper()).




### 詳細(Details)
See: [here](../doxygen/classJavaCallWrapper.html) for details

---
## <a name="notMiqrSFZ" id="notMiqrSFZ">JavaCallArguments</a>

### 概要(Summary)
JavaCalls クラス用の補助クラス(StackObjクラス) (See: [here](no3059iJu.html) for details).

JavaCalls による Java コード呼び出し時に「引数」を格納しておくコンテナクラス.

(JavaCalls::call_helper() は引数として methodOop と JavaCallArguments を受けとる. 
 コメントによると, var-args より速くて安全で便利, とのこと.)


```cpp
    ((cite: hotspot/src/share/vm/runtime/javaCalls.hpp))
    // Encapsulates arguments to a JavaCall (faster, safer, and more convenient than using var-args)
    class JavaCallArguments : public StackObj {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

(主に JavaCalls による呼び出し箇所)

(内部的に JavaCalls を使っている関係上, Exceptions クラス等でも JavaCallArguments は使用されている)




### 詳細(Details)
See: [here](../doxygen/classJavaCallArguments.html) for details

---
## <a name="noVMAU0HMM" id="noVMAU0HMM">SignatureChekker</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

メソッドの型情報(methodOop の signature 情報)と実際の引数の型が合っているかどうかチェックする.

(ところでこのクラス名は typo?? #TODO)


```cpp
    ((cite: hotspot/src/share/vm/runtime/javaCalls.cpp))
    class SignatureChekker : public SignatureIterator {
```

### 使われ方(Usage)
JavaCallArguments::verify() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classSignatureChekker.html) for details

---
