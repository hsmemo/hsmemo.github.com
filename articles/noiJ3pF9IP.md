---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ネイティブコードの処理中 ： CVMI 関数の呼び出し時 (JVM_*)
---
[Up](nouSkNo9hy.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ネイティブコードの処理中 ： CVMI 関数の呼び出し時 (JVM_*)

--- 
## 概要(Summary)
CVMI 関数の呼び出し時には, 明示的に SafepointSynchronize::_state のチェックが行われる.
この時点で Safepoint が開始されていた場合, そのスレッドはその場で停止する.

より具体的に言うと, CVMI 関数は定義時に以下のマクロを用いて定義されている (See: ...).

  * JVM_ENTRY マクロ (または JVM_ENTRY_NO_ENV マクロ, JVM_QUICK_ENTRY マクロ)
  * JVM_END マクロ

これらのマクロが生成するコードから ThreadInVMfromNative のコンストラクタ/デストラクタが呼び出され,
その中で SafepointSynchronize::_state の値を確認している.

## 備考(Notes)
非常に簡単な CVMI 関数用に, Safepoint チェックを行わない JVM_LEAF というマクロも用意されている.


## 処理の流れ (概要)(Execution Flows : Summary)
### JVM_ENTRY マクロ ~ JVM_END マクロ
```
JVM_ENTRY マクロ
-> ThreadInVMfromNative::ThreadInVMfromNative()
   -> (See: [here](no8p2E6iLf.html) for details)
```

```
JVM_END マクロ
(JVM_ENTRY マクロで宣言されていた ThreadInVMfromNative のデストラクタが呼ばれる)
-> ThreadInVMfromNative::~ThreadInVMfromNative()
   -> (See: [here](no8p2E6iLf.html) for details)
```


### JVM_ENTRY_NO_ENV マクロ ~ JVM_END マクロ
```
JVM_ENTRY_NO_ENV マクロ
-> ThreadInVMfromNative::ThreadInVMfromNative()
   -> (See: [here](no8p2E6iLf.html) for details)
```

```
JVM_END マクロ
-> (同上)
```


### JVM_QUICK_ENTRY マクロ ~ JVM_END マクロ
```
JVM_ENTRY_NO_ENV マクロ
-> ThreadInVMfromNative::ThreadInVMfromNative()
   -> (See: [here](no8p2E6iLf.html) for details)
```

```
JVM_END マクロ
-> (同上)
```


### JVM_LEAF マクロ ~ JVM_END マクロ
```
(Safepoint チェック処理はない)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### JVM_ENTRY マクロ
See: [here](no788267d.html) for details
### __ENTRY マクロ
See: [here](no7882ScE.html) for details
### JVM_END マクロ
See: [here](no7882txX.html) for details

### JVM_ENTRY_NO_ENV マクロ
See: [here](no7882HGk.html) for details

### JVM_QUICK_ENTRY マクロ
See: [here](no7882UQq.html) for details
### __QUICK_ENTRY マクロ
See: [here](no31977Dqo.html) for details

### JVM_LEAF マクロ
See: [here](no7882gnR.html) for details
### __LEAF マクロ
See: [here](no7882TdL.html) for details






