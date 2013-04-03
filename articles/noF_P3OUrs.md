---
layout: default
title: 同期排他処理に関する高レベル中間語(Ideal)クラス (BoxLockNode, FastLockNode, FastUnlockNode)
---
[Top](../index.html)

#### 同期排他処理に関する高レベル中間語(Ideal)クラス (BoxLockNode, FastLockNode, FastUnlockNode)

これらは, C2 JIT Compiler 用の高レベル中間語を表すクラス.
より具体的に言うと, 同期排他処理 (その中でも fast-path のロック/アンロック処理) を表すクラス.


### クラス一覧(class list)

  * [BoxLockNode](#noBFpilVHH)
  * [FastLockNode](#noosq3r_B1)
  * [FastUnlockNode](#noYv663zxg)


---
## <a name="noBFpilVHH" id="noBFpilVHH">BoxLockNode</a>

### 概要(Summary)
Node クラスの具象サブクラスの1つ.

fast-path の処理で使用される BasicObjectLock のアドレスを出力する Node
(= そのロック処理でスタックフレーム上のどの BasicObjectLock を使用するかを表す Node).


```
    ((cite: hotspot/src/share/vm/opto/locknode.hpp))
    //------------------------------BoxLockNode------------------------------------
    class BoxLockNode : public Node {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* GraphKit::shared_lock()
* Parse::load_interpreter_state()

### 内部構造(Internal structure)
入力ノードは control input のみ. そして control input は常に RootNode.


```
    ((cite: hotspot/src/share/vm/opto/locknode.cpp))
    BoxLockNode::BoxLockNode( int slot ) : Node( Compile::current()->root() ),
                                           _slot(slot), _is_eliminated(false) {
      init_class_id(Class_BoxLock);
      init_flags(Flag_rematerialize);
      OptoReg::Name reg = OptoReg::stack2reg(_slot);
      _inmask.Insert(reg);
    }
```




### 詳細(Details)
See: [here](../doxygen/classBoxLockNode.html) for details

---
## <a name="noosq3r_B1" id="noosq3r_B1">FastLockNode</a>

### 概要(Summary)
CmpNode クラスの具象サブクラスの1つ.

fast-path でのロック確保処理を表す
(なお fast-path でのロック確保処理は失敗することがあるので成否を表すために CmpNode のサブクラスとなっている).


```
    ((cite: hotspot/src/share/vm/opto/locknode.hpp))
    //------------------------------FastLockNode-----------------------------------
    class FastLockNode: public CmpNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* GraphKit::shared_lock()
* Parse::load_interpreter_state()

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. 
なお, コンストラクタで指定することにより 0 以外の control input を設定できる. ただし現状では 0 しか指定されていない.

その他の 2つは以下の通り.

* 2番目の入力Node : ロック確保対象の oop
* 3番目の入力Node : 使用する BasicObjectLock (を表す BoxLockNode)




### 詳細(Details)
See: [here](../doxygen/classFastLockNode.html) for details

---
## <a name="noYv663zxg" id="noYv663zxg">FastUnlockNode</a>

### 概要(Summary)
CmpNode クラスの具象サブクラスの1つ.

fast-path でのロック解放処理を表す
(なお fast-path ではロックが解放できないことがあるので成否を表すために CmpNode のサブクラスとなっている).


```
    ((cite: hotspot/src/share/vm/opto/locknode.hpp))
    //------------------------------FastUnlockNode---------------------------------
    class FastUnlockNode: public CmpNode {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
PhaseMacroExpand::expand_unlock_node() 内で(のみ)生成されている.

### 内部構造(Internal structure)
(control input も含めて) 3つの入力ノードを持つ. 

それぞれの入力の意味は以下の通り.

* 1番目の入力Node : control input
* 2番目の入力Node : ロック解放対象の oop
* 3番目の入力Node : 使用する BasicObjectLock (を表す BoxLockNode)




### 詳細(Details)
See: [here](../doxygen/classFastUnlockNode.html) for details

---
