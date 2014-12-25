---
layout: default
title: vframeArray クラス関連のクラス (vframeArrayElement, vframeArray)
---
[Top](../index.html)

#### vframeArray クラス関連のクラス (vframeArrayElement, vframeArray)

これらは, JIT Compiler 用のクラス.
より具体的に言うと, 脱最適化処理で使用される補助クラス (See: [here](no3420xYb.html) for details).


### クラス一覧(class list)

  * [vframeArray](#nocNLsML40)
  * [vframeArrayElement](#nokk4OAt4F)


---
## <a name="nocNLsML40" id="nocNLsML40">vframeArray</a>

### 概要(Summary)
JIT Compiler 用のクラス (より正確に言うと, 脱最適化処理で使用される補助クラス).

脱最適化対象となったスタックフレーム内の情報を記録しておくためのクラス.
これらの情報は, 脱最適化によって Interpreter フレームが構築された後, 
Interpreter フレーム用のフォーマットでスタックフレーム内に書き戻される.

(なお, このクラスは CHeapObj クラスだが, 本質的には ResourceObj クラスにしても問題ない.
後から中身を参照できるとデバッグ時に便利なので CHeapObj となっているだけ).


```cpp
    ((cite: hotspot/src/share/vm/runtime/vframeArray.hpp))
    // this can be a ResourceObj if we don't save the last one...
    // but it does make debugging easier even if we can't look
    // at the data in each vframeElement
    
    class vframeArray: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 JavaThread オブジェクトの _vframe_array_head フィールド

* 各 JavaThread オブジェクトの _vframe_array_last フィールド
  
  (ただし, こちらはデバッグ用のフィールド)

#### 生成箇所(where its instances are created)
vframeArray::allocate() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
Deoptimization::fetch_unroll_info_helper()
-> Deoptimization::create_vframeArray()
   -> vframeArray::allocate()
```

#### 削除箇所(where its instances are deleted)
以下の箇所で(のみ)削除されている.

* Deoptimization::cleanup_deopt_info()
  
  (より正確に言うと, JavaThread::_vframe_array_last フィールドの vframeArray が削除され, 
  JavaThread::_vframe_array_head フィールドの vframeArray が JavaThread::_vframe_array_last フィールドに移される)

* JavaThread::~JavaThread()
  
  (より正確に言うと, JavaThread::_vframe_array_last フィールドの vframeArray が削除される)

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

(_elements フィールドの大きさは可変長. 必要な個数の vframeArrayElement オブジェクトがここに格納されている)


```cpp
    ((cite: hotspot/src/share/vm/runtime/vframeArray.hpp))
      // Here is what a vframeArray looks like in memory
    
      /*
          fixed part
            description of the original frame
            _frames - number of vframes in this array
            adapter info
            callee register save area
          variable part
            vframeArrayElement   [ 0 ]
            ...
            vframeArrayElement   [_frames - 1]
    
      */
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/vframeArray.hpp))
      JavaThread*                  _owner_thread;
      vframeArray*                 _next;
      frame                        _original;          // the original frame of the deoptee
      frame                        _caller;            // caller of root frame in vframeArray
      frame                        _sender;
    
      Deoptimization::UnrollBlock* _unroll_block;
      int                          _frame_size;
    
      int                          _frames; // number of javavframes in the array (does not count any adapter)
    
      intptr_t                     _callee_registers[RegisterMap::reg_count];
      unsigned char                _valid[RegisterMap::reg_count];
    
      vframeArrayElement           _elements[1];   // First variable section.
```




### 詳細(Details)
See: [here](../doxygen/classvframeArray.html) for details

---
## <a name="nokk4OAt4F" id="nokk4OAt4F">vframeArrayElement</a>

### 概要(Summary)
vframeArray クラス用の補助クラス.

脱最適化対象となったスタックフレーム内の情報を記録しておくためのクラス.
1つの vframeArrayElement オブジェクトが 1つの (vframe 的な意味での) スタックフレームに対応する
(つまり Java レベルのメソッドと 1対1対応するような論理的なスタックフレームに対応. 
 インライン展開された場合は実際のスタックフレームとは 1対1対応しない).


```cpp
    ((cite: hotspot/src/share/vm/runtime/vframeArray.hpp))
    // A vframeArrayElement is an element of a vframeArray. Each element
    // represent an interpreter frame which will eventually be created.
    
    class vframeArrayElement : public _ValueObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 vframeArray オブジェクトの _elements フィールドに(のみ)格納されている.

(正確には, このフィールドは vframeArrayElement の配列を格納するフィールド.
この中に, その vframeArray 内で使用される全ての vframeArrayElement オブジェクトが格納されている)

(なお, 宣言では vframeArrayElement[1] となっているが, 実際の大きさは異なる(可変長)).

#### 生成箇所(where its instances are created)
配列用のメモリ領域は vframeArray::allocate() 内で(のみ)確保されている. 

そのメモリ領域中に個別の vframeArrayElement オブジェクトを書き込む作業は vframeArrayElement::fill_in() 内で(のみ)行われている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
vframeArray::allocate()
-> vframeArray::fill_in()
   -> vframeArrayElement::fill_in()
```




### 詳細(Details)
See: [here](../doxygen/classvframeArrayElement.html) for details

---
