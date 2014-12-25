---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 権限 (Capability) ： GetPotentialCapabilities(), AddCapabilities(), RelinquishCapabilities(), GetCapabilities() の処理  
---
[Up](noqskFx1QZ.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 権限 (Capability) ： GetPotentialCapabilities(), AddCapabilities(), RelinquishCapabilities(), GetCapabilities() の処理  

--- 
## 概要(Summary)
capability 関係は jvmtiCapabilities 構造体および JvmtiManageCapabilities クラスで管理している.

  * jvmtiCapabilities 構造体
    
    確保可能な (あるいは確保している) capability を表すデータ構造.

  * JvmtiManageCapabilities クラス
    
    capability(権限)管理用の関数を納めた名前空間(AllStatic クラス).
    jvmtiCapabilities 構造体も管理対象の1つ.

JvmtiManageCapabilities は, 
確保可能な capabilities を管理するための 4つの jvmtiCapabilities フィールドを持つ
(これらはそれぞれ「onload 時でないと取得できないかどうか」「最大1つの environment までしか取得できないかどうか」という分類に基づく.
 なお, この4つは排他的(disjoint)になるようにしている(= 重複する capability が存在しないようにしている), とのこと).

  * いつでも取れる capabilities

    JvmtiManageCapabilities::always_capabilities フィールド

  * OnLoad 時にしか取れない capabilities

    JvmtiManageCapabilities::onload_capabilities フィールド

  * いつでも取れるが 最大1つの environment までしか取得できない capabilities

    JvmtiManageCapabilities::always_solo_capabilities フィールド

  * OnLoad 時にしか取れず, かつ 最大1つの environment までしか取得できない capabilities

    JvmtiManageCapabilities::onload_solo_capabilities フィールド


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiManageCapabilities.hpp))
      // these four capabilities sets represent all potentially
      // available capabilities.  They are disjoint, covering
      // the four cases: (OnLoad vs OnLoad+live phase) X
      // (one environment vs any environment).
      static jvmtiCapabilities always_capabilities;
      static jvmtiCapabilities onload_capabilities;
      static jvmtiCapabilities always_solo_capabilities;
      static jvmtiCapabilities onload_solo_capabilities;
```


また, JvmtiManageCapabilities は, 
現在確保している capabilities を管理するための 3つの jvmtiCapabilities フィールドを持つ.

  * always_solo_capabilities のうち, 現在誰も取得指定していないもの (この値は実行時に変わっていく)
   
    JvmtiManageCapabilities::always_solo_remaining_capabilities フィールド
   
  * onload_solo_capabilities のうち, 現在誰も取得指定していないもの (この値は実行時に変わっていく)
   
    JvmtiManageCapabilities::onload_solo_remaining_capabilities フィールド
   
  * 1度でも取得されたことが有る capabilities (この値は実行時に変わっていく)
   
    JvmtiManageCapabilities::acquired_capabilities フィールド


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiManageCapabilities.hpp))
      // solo capabilities that have not been grabbed
      static jvmtiCapabilities always_solo_remaining_capabilities;
      static jvmtiCapabilities onload_solo_remaining_capabilities;
    
      // all capabilities ever acquired
      static jvmtiCapabilities acquired_capabilities;
```


## 備考(Notes)
各 capabilities の確保可能時期などを示す 4つの jvmtiCapabilities の値は, 
GetEnv() 時の大域的な初期化処理で決定される (See: [here](no30592Ee.html) for details).

詳細は以下の関数(後述)を参照.

  * JvmtiManageCapabilities::init_always_capabilities()
  * JvmtiManageCapabilities::init_onload_capabilities()
  * JvmtiManageCapabilities::init_always_solo_capabilities()
  * JvmtiManageCapabilities::init_onload_solo_capabilities()

(ただし, これは初期値である模様.
 onload 系のものであっても, 一度でも AddCapabilities に成功すれば always に移される
 (See: JvmtiManageCapabilities::add_capabilities()).
 そして, たとえ後で RelinquishCapabilities しても always に入ったまま)

(onload 系のものは有効にするといろいろと性能的にオーバーヘッドがかかるが,
 それらは always を見て決めているようなので,
 結局は一度でも有効にしたら使わなくてもオーバーヘッドはかかる模様)

## 備考(Notes)
AddCapabilities() や RelinquishCapabilities() の処理で capability の取得状況を変更した際には,
JvmtiManageCapabilities::update() で対応する JvmtiExport::can_* の値を変更している.

また, JvmtiManageCapabilities::update() の中では,
onload 系の capability が取得された場合に (それに支障になるような最適化をオフにするために)
いくつかのコマンドラインオプションの値を変更する, といった処理も行われている.
この際には, 現在実際に取得している capabilities ではなく, 
潜在的に取得可能な capabilities (というか具体的には always_capabilities と always_solo_capabilities) を元に計算する.
このため基本的に最適化機能を切る方向にしか update されない
(AddCapabilities() した後で RelinquishCapabilities() で手放しても最適化は無効にされたまま).


## 備考(Notes)
JvmtiEnvBase オブジェクトも, 内部に _current_capabilities と _prohibited_capabilities というフィールドを持つ (アクセサメソッドは get_capabilities() 及び get_prohibited_capabilities()).

とはいえ, _prohibited_capabilities を更新しているのは
JvmtiEnvBase::record_first_time_class_file_load_hook_enabled() 内で
retransform を追加しているくらい?? (#TODO)

## 備考(Notes)
JVMTI の DisposeEnvironment() 関数の処理時にも
JvmtiManageCapabilities::relinquish_capabilities() が呼び出されている.


## 処理の流れ (概要)(Execution Flows : Summary)
### GetEnv() 時の大域的な初期化処理
```
(See: [here](no30592Ee.html) for details)
-> JvmtiEnvBase::globally_initialize()
   -> JvmtiManageCapabilities::initialize()
      -> JvmtiManageCapabilities::init_always_capabilities()
      -> JvmtiManageCapabilities::recompute_always_capabilities()
      -> JvmtiManageCapabilities::init_onload_capabilities()
      -> JvmtiManageCapabilities::init_always_solo_capabilities()
      -> JvmtiManageCapabilities::init_onload_solo_capabilities()
```

### GetPotentialCapabilities() の処理
```
JvmtiEnv::GetPotentialCapabilities()
-> JvmtiManageCapabilities::get_potential_capabilities()
```

### AddCapabilities() の処理
```
JvmtiEnv::AddCapabilities()
-> JvmtiManageCapabilities::add_capabilities()
   -> いろいろと削除したり追加したり
   -> JvmtiManageCapabilities::update()
```

### RelinquishCapabilities() の処理
```
JvmtiEnv::RelinquishCapabilities()
-> JvmtiManageCapabilities::relinquish_capabilities()
   -> いろいろと削除したり追加したり
   -> JvmtiManageCapabilities::update()
```

### GetCapabilities() の処理
```
JvmtiEnv::GetCapabilities()
-> JvmtiManageCapabilities::copy_capabilities()
```

### DisposeEnvironment() 時の処理
```
(See: [here](no2935HVZ.html) for details)
-> JvmtiEnvBase::env_dispose()
   -> JvmtiManageCapabilities::relinquish_capabilities()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiManageCapabilities::initialize()
See: [here](no79924wx.html) for details
### JvmtiManageCapabilities::init_always_capabilities()
See: [here](no2935bCh.html) for details
### JvmtiManageCapabilities::recompute_always_capabilities()
See: [here](no7992EPN.html) for details
### JvmtiManageCapabilities::init_onload_capabilities()
See: [here](no2935oMn.html) for details
### JvmtiManageCapabilities::init_always_solo_capabilities()
See: [here](no29351Wt.html) for details
### JvmtiManageCapabilities::init_onload_solo_capabilities()
See: [here](no2935Chz.html) for details

### JvmtiEnv::GetPotentialCapabilities()
See: [here](no29355QA.html) for details
### JvmtiManageCapabilities::get_potential_capabilities()
See: [here](no2935GbG.html) for details
### JvmtiEnvBase::get_capabilities()
See: [here](no29356Df.html) for details
### JvmtiEnvBase::get_prohibited_capabilities()
See: [here](no2935HOl.html) for details

### JvmtiEnv::AddCapabilities()
See: [here](no2935TlM.html) for details
### JvmtiManageCapabilities::add_capabilities()
See: [here](no2935gvS.html) for details
### JvmtiManageCapabilities::update()
See: [here](no2935UYr.html) for details

### JvmtiEnv::RelinquishCapabilities()
See: [here](no2935hix.html) for details
### JvmtiManageCapabilities::relinquish_capabilities()
See: [here](no2935TsA.html) for details

### JvmtiEnv::GetCapabilities()
See: [here](no2935g2G.html) for details
### JvmtiManageCapabilities::copy_capabilities()
See: [here](no2935tAN.html) for details






