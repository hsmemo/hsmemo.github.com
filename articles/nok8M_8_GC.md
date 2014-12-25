---
layout: default
title: AttachListener クラス関連のクラス (AttachListener, AttachOperation)
---
[Top](../index.html)

#### AttachListener クラス関連のクラス (AttachListener, AttachOperation)

これらは, 保守運用機能のためのクラス.
より具体的に言うと, "Dynamic Attach" 機能のためのクラス (See: [here](no3026gMG.html) for details).


### クラス一覧(class list)

  * [AttachListener](#nohz3gNl5n)
  * [AttachOperation](#no_7x1Ugob)


---
## <a name="nohz3gNl5n" id="nohz3gNl5n">AttachListener</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する serviceability 機能からのみ使用される).

Dynamic Attach に関する機能を納めた名前空間(AllStatic クラス) (See: [here](no3026gMG.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/attachListener.hpp))
    class AttachListener: AllStatic {
```

### 内部構造(Internal structure)
実際の処理 (特にクライアントから要求を受け取る処理) は OS 依存の補助クラスに任せている箇所が多い
(See: LinuxAttachListener, SolarisAttachListener, Win32AttachListener).




### 詳細(Details)
See: [here](../doxygen/classAttachListener.html) for details

---
## <a name="no_7x1Ugob" id="no_7x1Ugob">AttachOperation</a>

### 概要(Summary)
Dynamic Attach 機能の client による HotSpot への要求内容を表すクラス (See: [here](no3026gMG.html) for details).

1つの AttachOperation オブジェクトが client から送られてきたリクエスト1個に対応する. 


```cpp
    ((cite: hotspot/src/share/vm/services/attachListener.hpp))
    class AttachOperation: public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

### 使われ方(Usage)
client からの要求は AttachListener::dequeue() メソッドで取り出される.
この返値が AttachOperation オブジェクトとなっている.

```cpp
    ((cite: hotspot/src/share/vm/services/attachListener.hpp))
      // dequeue the next operation
      static AttachOperation* dequeue();
```




### 詳細(Details)
See: [here](../doxygen/classAttachOperation.html) for details

---
