---
layout: default
title: Relocator クラス関連のクラス (RelocatorListener, Relocator, 及びそれらの補助クラス(ChangeItem, ChangeWiden, ChangeJumpWiden, ChangeSwitchPad))
---
[Top](../index.html)

#### Relocator クラス関連のクラス (RelocatorListener, Relocator, 及びそれらの補助クラス(ChangeItem, ChangeWiden, ChangeJumpWiden, ChangeSwitchPad))

これらは, バイトコードを書き換える処理用のユーティリティ・クラス.


### クラス一覧(class list)

  * [Relocator](#no3OogJ7we)
  * [RelocatorListener](#noKvKrboMI)
  * [ChangeItem](#nokFIffsh3)
  * [ChangeWiden](#no-EI5h2HU)
  * [ChangeJumpWiden](#nokXWwM8f4)
  * [ChangeSwitchPad](#noxayIP8Oi)


---
## <a name="no3OogJ7we" id="no3OogJ7we">Relocator</a>

### 概要(Summary)
何らかのバイトコード書き換え処理で使用される一時オブジェクト(ResourceObjクラス).

(なお, このクラスは ResourceObj クラスだが局所変数としてのみ生成されている)

バイトコード中の命令を書き換えるためのメソッドを提供している.

(なおバイトコードを書き換えた場合は, 
 分岐命令の飛び先や例外テーブルの情報, LineNumberTable, StackMapTable, etc も合わせて変更しないとまずいが, 
 それも行ってくれる)


```
    ((cite: hotspot/src/share/vm/runtime/relocator.hpp))
    class Relocator : public ResourceObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. コード中で Relocator 型の局所変数を宣言する (コンストラクタ引数で変更対象のメソッドを指定する).

   (なお変更時にコールバックして欲しい場合は, コンストラクタ引数で RelocatorListener オブジェクトも渡しておく)

2. 変更したい箇所と新しい命令を指定して Relocator::insert_space_at() を呼ぶと, 指定箇所のバイトコード命令を書き換えてくれる

   (新旧のバイトコードの長さに応じて, 
   分岐命令の飛び先や例外テーブルの情報, LineNumberTable, etc も合わせて変更してくれる).

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* GenerateOopMap::expand_current_instr()
* VM_RedefineClasses::rewrite_cp_refs_in_method()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

```
* GenerateOopMap::expand_current_instr() の呼び出しパス

  GenerateOopMap::compute_map()
  -> GenerateOopMap::do_interpretation()
     -> GenerateOopMap::rewrite_refval_conflicts()
        -> GenerateOopMap::rewrite_refval_conflict()
           -> GenerateOopMap::rewrite_refval_conflict_inst()
              -> GenerateOopMap::rewrite_load_or_store()
                 -> GenerateOopMap::expand_current_instr()

* VM_RedefineClasses::rewrite_cp_refs_in_method() の呼び出しパス
  
  VM_RedefineClasses::doit_prologue()
  -> VM_RedefineClasses::load_new_class_versions()
     -> VM_RedefineClasses::merge_cp_and_rewrite()
        -> VM_RedefineClasses::rewrite_cp_refs()
           -> VM_RedefineClasses::rewrite_cp_refs_in_methods()
              -> VM_RedefineClasses::rewrite_cp_refs_in_method()
```

### 内部構造(Internal structure)
バイトコード中の命令に関しては, 変更が必要な箇所があまり自明でなく, 
1つの命令を変更するとそれが原因となって他の命令の変更が必要になったりする.

そこで, 内部では「ChangeItem というオブジェクトの配列」として変更予定箇所を管理している. 
変更処理ではこれらの ChangeItem オブジェクトが表す変更内容を次々と実行していき
(その際に他の箇所の変更が必要になれば ChangeItem オブジェクトが新たに追加される), 
配列が空になったら処理が完了する
(See: Relocator::handle_code_changes()).




### 詳細(Details)
See: [here](../doxygen/classRelocator.html) for details

---
## <a name="noKvKrboMI" id="noKvKrboMI">RelocatorListener</a>

### 概要(Summary)
Relocator クラスでの書き換え処理中に使用される一時オブジェクト(StackObjクラス) (の基底クラス).

Relocator がコードを変更した際に, 
その変更箇所に関する情報を通知(コールバック)してもらって何らかの処理を行えるようにするためのクラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/relocator.hpp))
    // Callback object for code relocations
    class RelocatorListener : public StackObj {
```

### 使われ方(Usage)
使用する際には, relocated() メソッドをオーバーライドしたサブクラスを作ればいい.


```
    ((cite: hotspot/src/share/vm/runtime/relocator.hpp))
      virtual void relocated(int bci, int delta, int new_method_size) = 0;
```

Relocator オブジェクトの生成時にコンストラクタ引数で RelocatorListener オブジェクトも渡しておくと, 
バイトコード書き換え時にそのオブジェクトの RelocatorListener::relocated() メソッドが呼び出される.




### 詳細(Details)
See: [here](../doxygen/classRelocatorListener.html) for details

---
## <a name="nokFIffsh3" id="nokFIffsh3">ChangeItem</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス (の基底クラス).

Relocator の処理において, 変更が必要なバイトコード箇所を管理するための一時オブジェクト(ResourceObjクラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/relocator.cpp))
    // Encapsulates a code change request. There are 3 types.
    // General instruction, jump instruction, and table/lookup switches
    //
    class ChangeItem : public ResourceObj {
```



### 詳細(Details)
See: [here](../doxygen/classChangeItem.html) for details

---
## <a name="no-EI5h2HU" id="no-EI5h2HU">ChangeWiden</a>

### 概要(Summary)
ChangeItem クラスの具象サブクラスの1つ.

このクラスは, Relocator::insert_space_at() で指定されたバイトコードを表す.


```
    ((cite: hotspot/src/share/vm/runtime/relocator.cpp))
    class ChangeWiden : public ChangeItem {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Relocator オブジェクトの _changes フィールドに(のみ)格納されている
(ただし, ResourceObjクラスなので一時的なオブジェクト).

(正確には, このフィールドは ChangeItem の GrowableArray を格納するフィールド.
この中に, 変更処理の内容を表す全ての ChangeItem オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
Relocator::insert_space_at() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classChangeWiden.html) for details

---
## <a name="nokXWwM8f4" id="nokXWwM8f4">ChangeJumpWiden</a>

### 概要(Summary)
ChangeItem クラスの具象サブクラスの1つ.

このクラスは, 連鎖的に修正が必要になった分岐命令(if*,goto,jsr)を表す.


```
    ((cite: hotspot/src/share/vm/runtime/relocator.cpp))
    class ChangeJumpWiden : public ChangeItem {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Relocator オブジェクトの _changes フィールドに(のみ)格納されている
(ただし, ResourceObjクラスなので一時的なオブジェクト).

(正確には, このフィールドは ChangeItem の GrowableArray を格納するフィールド.
この中に, 変更処理の内容を表す全ての ChangeItem オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
Relocator::push_jump_widen() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classChangeJumpWiden.html) for details

---
## <a name="noxayIP8Oi" id="noxayIP8Oi">ChangeSwitchPad</a>

### 概要(Summary)
ChangeItem クラスの具象サブクラスの1つ.

このクラスは, 連鎖的に修正が必要になった分岐命令(*switch)を表す.


```
    ((cite: hotspot/src/share/vm/runtime/relocator.cpp))
    class ChangeSwitchPad : public ChangeItem {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Relocator オブジェクトの _changes フィールドに(のみ)格納されている
(ただし, ResourceObjクラスなので一時的なオブジェクト).

(正確には, このフィールドは ChangeItem の GrowableArray を格納するフィールド.
この中に, 変更処理の内容を表す全ての ChangeItem オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
Relocator::change_jumps() 内で(のみ)生成されている.





### 詳細(Details)
See: [here](../doxygen/classChangeSwitchPad.html) for details

---
