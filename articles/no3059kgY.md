---
layout: default
title: Method に関する処理 ： ダイナミックディスパッチ(dynamic dispatch)処理 
---
[Up](nojMOQfYh-.html) [Top](../index.html)

#### Method に関する処理 ： ダイナミックディスパッチ(dynamic dispatch)処理 

--- 
## 概要(Summary)
ダイナミックディスパッチ用のテーブルは各 instanceKlass オブジェクト内に格納されている.
これらのテーブルは vtable および itable と呼ばれており, それぞれ invokevirtual および invokeinterface 用 (See: [here](nok3cQYg2G.html) for details).

vtable と itable はクラスファイルのパース時に生成され, 実際の呼び出し時に参照される.

### vtable/itable の構築処理
クラスファイルのパースによる instanceKlassOop の作成時に,
ClassFileParser::parseClassFile() 内で以下のようにして領域が確保される. (See: [here](no2114rPX.html) for details)

  1. パースの終了後に, klassVtable::compute_vtable_size_and_num_mirandas() を呼んで vtable/itable の大きさを計算
  2. 上記サイズを元にして oopFactory::new_instanceKlass() を呼び出し, klassOop を確保
  3. klassItable::setup_itable_offset_table() を呼び出して, (#TODO)

その後のリンク処理時に, instanceKlass::link_class_impl() 内でテーブルの中身が書き込まれる. (See: [here](no3059xqe.html) for details)

  4. klassVtable::initialize_vtable() で vtable が生成される
  5. klassItable::initialize_itable() で itable が生成される
  

### 実際のダイナミックディスパッチ処理

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table9282xHE -->
| Type | Related Links |
|---|---|
| Runtime(C++ コード) からのダイナミックディスパッチ | (See: [here](no3059iJu.html) for details) |
| Template Interpreter からのダイナミックディスパッチ | (See: [here](no3059-tw.html) for details), (See: [here](no3059L42.html) for details) |
| CPP Interpreter からのダイナミックディスパッチ | (#Under Construction) |
| C1 JIT 生成コードからのダイナミックディスパッチ | (#Under Construction) |
| C2 JIT 生成コードからのダイナミックディスパッチ | (See: [here](no3059xjq.html) for details) |
| Shark JIT 生成コードからのダイナミックディスパッチ | (#Under Construction) |
| JNI からのダイナミックディスパッチ | (See: [here](no3059-0k.html) for details) |
<!-- END RECEIVE ORGTBL table9282xHE -->

<!-- 
#+ORGTBL: SEND table9282xHE orgtbl-to-gfm :no-escape t
| Type                                                | Related Links                          |
|-----------------------------------------------------+----------------------------------------|
| Runtime(C++ コード) からのダイナミックディスパッチ  | (See: [here](no3059iJu.html) for details)                     |
| Template Interpreter からのダイナミックディスパッチ | (See: [here](no3059-tw.html) for details), (See: [here](no3059L42.html) for details) |
| CPP Interpreter からのダイナミックディスパッチ      | (#Under Construction)                  |
| C1 JIT 生成コードからのダイナミックディスパッチ     | (#Under Construction)                  |
| C2 JIT 生成コードからのダイナミックディスパッチ     | (See: [here](no3059xjq.html) for details)                     |
| Shark JIT 生成コードからのダイナミックディスパッチ  | (#Under Construction)                  |
| JNI からのダイナミックディスパッチ                  | (See: [here](no3059-0k.html) for details)                     |
-->







