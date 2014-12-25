---
layout: default
title: BasicLock クラスおよび BasicObjectLock クラス (BasicLock, BasicObjectLock)
---
[Top](../index.html)

#### BasicLock クラスおよび BasicObjectLock クラス (BasicLock, BasicObjectLock)

これらは, 同期排他処理 (monitorenter/monitorexit 命令, synchronized 修飾子) のためのクラス
(See: [here](no2114NIs.html) for details).


### クラス一覧(class list)

  * [BasicLock](#noehPwCdVp)
  * [BasicObjectLock](#nowq2rPXYz)


---
## <a name="noehPwCdVp" id="noehPwCdVp">BasicLock</a>

### 概要(Summary)
BasicObjectLock クラス用の補助クラス.

ロック対象のオブジェクトから退避した mark フィールド(markOopDesc)を格納しておくためのクラス
(ロック処理では mark フィールドを書き換えてしまうため, 元の値を記録しておく必要がある).


```cpp
    ((cite: hotspot/src/share/vm/runtime/basicLock.hpp))
    class BasicLock VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (他にもある?? #TODO).

* 各 BasicObjectLock オブジェクトの _lock フィールド
* 各 ObjectLocker オブジェクトの _lock フィールド

#### 生成箇所(where its instances are created)
* (BasicObjectLock クラスの _lock フィールドは, ポインタ型ではなく実体なので,
   BasicObjectLock オブジェクトの生成時に一緒に生成される)

* (ObjectLocker クラスの _lock フィールドは, ポインタ型ではなく実体なので,
   ObjectLocker オブジェクトの生成時に一緒に生成される)

### 内部構造(Internal structure)
内部には以下のフィールドが1つあるだけ.


```cpp
    ((cite: hotspot/src/share/vm/runtime/basicLock.hpp))
      volatile markOop _displaced_header;
```




### 詳細(Details)
See: [here](../doxygen/classBasicLock.html) for details

---
## <a name="nowq2rPXYz" id="nowq2rPXYz">BasicObjectLock</a>

### 概要(Summary)
同期排他処理 (monitorenter 命令, monitorexit 命令) のためのクラス (See: [here](no2114NIs.html) for details).

あるスレッドがあるメソッド内で取得したロックを記録しておくためのクラス.
メソッド内でのロック解放漏れがないかどうか(IllegalMonitorStateException)のチェック処理で使用される.
また, ロック処理が書き換えた mark フィールド(markOopDesc)の待避場所としても使用される
(ただし, この中に mark フィールドが待避されているのは stack-locked 状態の場合のみ).

なお, BasicObjectLock オブジェクトは対応するスタックフレーム内に埋め込まれている
(コメントでは "interpreter frame" と書かれているが, 
 JIT コンパイルされたメソッドの場合もスタックフレーム内に埋め込まれている).


```cpp
    ((cite: hotspot/src/share/vm/runtime/basicLock.hpp))
    // A BasicObjectLock associates a specific Java object with a BasicLock.
    // It is currently embedded in an interpreter frame.
    
    // Because some machines have alignment restrictions on the control stack,
    // the actual space allocated by the interpreter may include padding words
    // after the end of the BasicObjectLock.  Also, in order to guarantee
    // alignment of the embedded BasicLock objects on such machines, we
    // put the embedded BasicLock at the beginning of the struct.
    
    class BasicObjectLock VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
スタックフレーム内に(のみ)格納されている
(See: [here](nodNTDfmy6.html) for details) (See: ...).

#### 生成箇所(where its instances are created)
インタープリタの場合は, 未使用の BasicObjectLock がなくなるとスタックフレームを拡張して確保される
(See: TemplateTable::monitorenter(), ).

JIT コンパイルされたメソッドの場合は, スタックフレームの確保時に同時に生成される.

### 内部構造(Internal structure)
内部には, ロック対象のオブジェクト(oop), 
及びそのオブジェクトの mark フィールドを待避した BasicLock オブジェクトを保持している


```cpp
    ((cite: hotspot/src/share/vm/runtime/basicLock.hpp))
      BasicLock _lock;                                    // the lock, must be double word aligned
      oop       _obj;                                     // object holds the lock;
```




### 詳細(Details)
See: [here](../doxygen/classBasicObjectLock.html) for details

---
