---
layout: default
title: ScopeDesc クラス関連のクラス (SimpleScopeDesc, ScopeDesc)
---
[Top](../index.html)

#### ScopeDesc クラス関連のクラス (SimpleScopeDesc, ScopeDesc)

これらは, JIT コンパイラが生成したコードについて, ソースコードレベルでのデバッグに必要な情報を覚えておくためのクラス.

なお, SimpleScopeDesc は ScopeDesc の簡易版.


### クラス一覧(class list)

  * [ScopeDesc](#noF6pjXI0V)
  * [SimpleScopeDesc](#no5SSDRqGp)


---
## <a name="noF6pjXI0V" id="noF6pjXI0V">ScopeDesc</a>

### 概要(Summary)
nmethod オブジェクトについて, ソースコードレベルでのデバッグに必要な情報を記録したクラス.

("method activation" と書かれているので, 1つの ScopeDesc がソースコードレベルでの 1つの関数フレームに対応する模様?? #TODO)


```cpp
    ((cite: hotspot/src/share/vm/code/scopeDesc.hpp))
    // ScopeDescs contain the information that makes source-level debugging of
    // nmethods possible; each scopeDesc describes a method activation
    
    class ScopeDesc : public ResourceObj {
```

### 使われ方(Usage)
コンストラクタ引数で nmethod とその中の pc(というか命令のアドレス) を指定する.
生成した ScopeDesc オブジェクトからは, 以下のような情報が取り出せる.

  * 対応する methodOop
  * 対応する bytecode の index (bci)
  * #TODO (GrowableArray<ScopeValue*>*   locals();        // Javaレベルでの各local variables が, compiled frame のどこに存在するかを示す？)
  * #TODO (GrowableArray<ScopeValue*>*   expressions();   // Javaレベルでの各operand stack 上の値 が, compiled frame のどこに存在するかを示す？？？)
  * #TODO (GrowableArray<MonitorValue*>* monitors();)
  * #TODO (GrowableArray<ScopeValue*>*   objects();)




### 詳細(Details)
See: [here](../doxygen/classScopeDesc.html) for details

---
## <a name="no5SSDRqGp" id="no5SSDRqGp">SimpleScopeDesc</a>

### 概要(Summary)
ScopeDesc の簡易版.

欲しい情報が methodOop と bci だけであれば高速なこちらのクラスも利用可能, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/code/scopeDesc.hpp))
    // SimpleScopeDesc is used when all you need to extract from
    // a given pc,nmethod pair is a methodOop and a bci. This is
    // quite a bit faster than allocating a full ScopeDesc, but
    // very limited in abilities.
    
    class SimpleScopeDesc : public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classSimpleScopeDesc.html) for details

---
