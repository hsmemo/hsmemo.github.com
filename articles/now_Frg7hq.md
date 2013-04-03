---
layout: default
title: CallGenerator 及びそのサブクラス (CallGenerator, InlineCallGenerator, WarmCallInfo, 及びそれらの補助クラス(ParseGenerator, DirectCallGenerator, DynamicCallGenerator, VirtualCallGenerator, LateInlineCallGenerator, WarmCallGenerator, PredictedCallGenerator, PredictedDynamicCallGenerator, UncommonTrapCallGenerator))
---
[Top](../index.html)

#### CallGenerator 及びそのサブクラス (CallGenerator, InlineCallGenerator, WarmCallInfo, 及びそれらの補助クラス(ParseGenerator, DirectCallGenerator, DynamicCallGenerator, VirtualCallGenerator, LateInlineCallGenerator, WarmCallGenerator, PredictedCallGenerator, PredictedDynamicCallGenerator, UncommonTrapCallGenerator))

これらは, C2 JIT Compiler 用のクラス.
より具体的に言うと, メソッド呼び出しに関する高レベル中間語(Ideal)の構築を行うクラス.


### クラス一覧(class list)

  * [CallGenerator](#noORNj6QKa)
  * [InlineCallGenerator](#nouxnQOB4V)
  * [WarmCallInfo](#nogr7bHDug)
  * [ParseGenerator](#noIXS0IDwe)
  * [DirectCallGenerator](#noAICV_wGN)
  * [DynamicCallGenerator](#noz-bbzved)
  * [VirtualCallGenerator](#nosqaDd5WV)
  * [LateInlineCallGenerator](#no_wVuhUWJ)
  * [WarmCallGenerator](#nolJZmNFWa)
  * [PredictedCallGenerator](#nohgmzQhGZ)
  * [PredictedDynamicCallGenerator](#noU1PKDurZ)
  * [UncommonTrapCallGenerator](#noqIvz7O-g)


---
## <a name="noORNj6QKa" id="noORNj6QKa">CallGenerator</a>

### 概要(Summary)
メソッド呼び出し処理用の Ideal を生成するための一時オブジェクト(ResourceObjクラス) (の基底クラス).
このクラスのサブクラスがそれぞれの種類のメソッド呼び出しを表す.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.hpp))
    //---------------------------CallGenerator-------------------------------------
    // The subclasses of this class handle generation of ideal nodes for
    // call sites and method entry points.
    
    class CallGenerator : public ResourceObj {
```

### 使われ方(Usage)
使用する際には, generate() メソッドをオーバーライドしたサブクラスを作ればいい.


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.hpp))
      // The given jvms has state and arguments for a call to my method.
      // Edges after jvms->argoff() carry all (pre-popped) argument values.
      //
      // Update the map with state and return values (if any) and return it.
      // The return values (0, 1, or 2) must be pushed on the map's stack,
      // and the sp of the jvms incremented accordingly.
      //
      // The jvms is returned on success.  Alternatively, a copy of the
      // given jvms, suitably updated, may be returned, in which case the
      // caller should discard the original jvms.
      //
      // The non-Parm edges of the returned map will contain updated global state,
      // and one or two edges before jvms->sp() will carry any return values.
      // Other map edges may contain locals or monitors, and should not
      // be changed in meaning.
      //
      // If the call traps, the returned map must have a control edge of top.
      // If the call can throw, the returned map must report has_exceptions().
      //
      // If the result is NULL, it means that this CallGenerator was unable
      // to handle the given call, and another CallGenerator should be consulted.
      virtual JVMState* generate(JVMState* jvms) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classCallGenerator.html) for details

---
## <a name="nouxnQOB4V" id="nouxnQOB4V">InlineCallGenerator</a>

### 概要(Summary)
CallGenerator クラスのサブクラスの1つ. 
このクラスはメソッド呼び出しをインライン展開する場合用.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.hpp))
    class InlineCallGenerator : public CallGenerator {
```




### 詳細(Details)
See: [here](../doxygen/classInlineCallGenerator.html) for details

---
## <a name="nogr7bHDug" id="nogr7bHDug">WarmCallInfo</a>

### 概要(Summary)
Compile クラス及び InlineTree クラス用の補助クラス.

メソッドのインライン展開処理で使用される一時オブジェクト(ResourceObjクラス).
インライン展開するかどうかの判断結果を格納している.

なお, インライン展開するかどうかは温度になぞらえて表現されている
('hot' であればインライン展開する, 'cold' であればしない, 
その中間の温度であれば状況に応じてインライン展開する.
ただし, develop オプションである InlineWarmCalls がセットされていない場合, 中間の温度は使用されない)


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.hpp))
    //---------------------------WarmCallInfo--------------------------------------
    // A struct to collect information about a given call site.
    // Helps sort call sites into "hot", "medium", and "cold".
    // Participates in the queueing of "medium" call sites for possible inlining.
    class WarmCallInfo : public ResourceObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
InlineTree::ok_to_inline() を呼ぶと, 
指定したメソッドについての判断結果を格納した WarmCallInfo オブジェクトが返される.

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* WarmCallInfo クラスの _always_hot フィールド (static フィールド)
  
  'hot' な WarmCallInfo オブジェクト

* WarmCallInfo クラスの _always_cold フィールド (static フィールド)
  
  'cold' な WarmCallInfo オブジェクト

* 各 Compile オブジェクトの _warm_calls フィールド
  
  'hot' と 'cold' の中間の WarmCallInfo オブジェクト

  (正確には, このフィールドは WarmCallInfo の線形リストを格納するフィールド.
  WarmCallInfo オブジェクトは _next フィールドで次の WarmCallInfo オブジェクトを指せる構造になっている.
  その Compile オブジェクト内で生成した WarmCallInfo オブジェクトは全てこのフィールドの線形リストに格納されている)
  
  (このリストには WarmCallGenerator::generate() で要素が追加される.
  その後, Compile::Inline_Warm() と Compile::Finish_Warm() 内で参照される)
  
  (なお, develop オプションである InlineWarmCalls がセットされていない場合には使用されない)

#### 生成箇所(where its instances are created)
* (WarmCallInfo クラスの _always_hot フィールドは, ポインタ型ではなく実体なので,
  初期段階で自動的に生成される)


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    WarmCallInfo WarmCallInfo::_always_hot(WarmCallInfo::MAX_VALUE(), WarmCallInfo::MAX_VALUE(),
                                           WarmCallInfo::MIN_VALUE(), WarmCallInfo::MIN_VALUE());
```

* (WarmCallInfo クラスの _always_cold フィールドは, ポインタ型ではなく実体なので,
  初期段階で自動的に生成される)


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    WarmCallInfo WarmCallInfo::_always_cold(WarmCallInfo::MIN_VALUE(), WarmCallInfo::MIN_VALUE(),
                                            WarmCallInfo::MAX_VALUE(), WarmCallInfo::MAX_VALUE());
```

* InlineTree::ok_to_inline()
  
  (develop オプションである InlineWarmCalls がセットされている場合にのみ生成される)

* Compile::call_generator() (局所変数として生成)




### 詳細(Details)
See: [here](../doxygen/classWarmCallInfo.html) for details

---
## <a name="noIXS0IDwe" id="noIXS0IDwe">ParseGenerator</a>

### 概要(Summary)
InlineCallGenerator クラスの具象サブクラスの1つ.

パース処理 (バイトコードから高レベル中間語(Ideal)への変換処理) を行う.

(なお, メソッドの中身を全て Ideal のグラフへと変換するので, インライン展開処理にも使用されている)


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    //-----------------------------ParseGenerator---------------------------------
    // Internal class which handles all direct bytecode traversal.
    class ParseGenerator : public InlineCallGenerator {
```

### 使われ方(Usage)
以下のファクトリメソッドが用意されており, その中で(のみ)生成されている.

* CallGenerator::for_inline()
* CallGenerator::for_osr()

そして, これらのファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis )
-> CallGenerator::for_inline()
-> CallGenerator::for_osr()

Parse::do_call()
-> Compile::call_generator()
   -> CallGenerator::for_method_handle_inline()
      -> Compile::call_generator()
         -> (同上)
   -> Compile::call_generator() (再帰呼び出し)
      -> (同上)
   -> CallGenerator::for_inline()
      -> Compile::call_generator()
         -> (同上)
```

### 内部構造(Internal structure)
実際のパース処理のほとんどは Parse クラスに丸投げしている.
(See: ParseGenerator::generate()).




### 詳細(Details)
See: [here](../doxygen/classParseGenerator.html) for details

---
## <a name="noAICV_wGN" id="noAICV_wGN">DirectCallGenerator</a>

### 概要(Summary)
CallGenerator クラスの具象サブクラスの1つ.

このクラスは, ダイナミックディスパッチ(receiver の型チェック)が不要なメソッド呼び出し用
(e.g. invokestatic, invokespecial, optimized virtual call).

(なお, ダイナミックディスパッチが不要なメソッド呼び出しはインライン展開が行われることもある.
このクラスはインライン展開しない場合用)


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    //---------------------------DirectCallGenerator------------------------------
    // Internal class which handles all out-of-line calls w/o receiver type checks.
    class DirectCallGenerator : public CallGenerator {
```

### 使われ方(Usage)
CallGenerator::for_direct_call() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_call()
-> Compile::call_generator()
   -> CallGenerator::for_direct_call()

PredictedCallGenerator::generate()
-> CallGenerator::for_direct_call()
   -> (同上)

PredictedDynamicCallGenerator::generate()
-> CallGenerator::for_direct_call()
   -> (同上)
```




### 詳細(Details)
See: [here](../doxygen/classDirectCallGenerator.html) for details

---
## <a name="noz-bbzved" id="noz-bbzved">DynamicCallGenerator</a>

### 概要(Summary)
CallGenerator クラスの具象サブクラスの1つ.

このクラスは, invokedynamic によるメソッド呼び出し用.

(なお, 最適化用のクラスである PredictedDynamicCallGenerator と併用されることもある (See: PredictedDynamicCallGenerator))


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    //---------------------------DynamicCallGenerator-----------------------------
    // Internal class which handles all out-of-line invokedynamic calls.
    class DynamicCallGenerator : public CallGenerator {
```

### 使われ方(Usage)
CallGenerator::for_dynamic_call() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_call()
-> Compile::call_generator()
   -> CallGenerator::for_dynamic_call()
```




### 詳細(Details)
See: [here](../doxygen/classDynamicCallGenerator.html) for details

---
## <a name="nosqaDd5WV" id="nosqaDd5WV">VirtualCallGenerator</a>

### 概要(Summary)
CallGenerator クラスの具象サブクラスの1つ.

このクラスは, ダイナミックディスパッチ(receiver の型チェック)が必要なメソッド呼び出し用
(e.g. invokevirtual, invokeinterface).

(なお, 最適化用のクラスである PredictedCallGenerator と併用されることもある (See: PredictedCallGenerator))


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    //--------------------------VirtualCallGenerator------------------------------
    // Internal class which handles all out-of-line calls checking receiver type.
    class VirtualCallGenerator : public CallGenerator {
```

### 使われ方(Usage)
CallGenerator::for_virtual_call() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_call()
-> Compile::call_generator()
   -> CallGenerator::for_virtual_call()
```




### 詳細(Details)
See: [here](../doxygen/classVirtualCallGenerator.html) for details

---
## <a name="no_wVuhUWJ" id="no_wVuhUWJ">LateInlineCallGenerator</a>

### 概要(Summary)
特殊な DirectCallGenerator クラス.

このクラスは, 種々の最適化が終わってからインライン展開した方がよいメソッド呼び出し用
(取りあえず DirectCallGenerator クラスとして Ideal を生成するが, 後でインライン展開される).


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    // Allow inlining decisions to be delayed
    class LateInlineCallGenerator : public DirectCallGenerator {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 Compile オブジェクトの _late_inlines フィールドに(のみ)格納されている.
 
(正確には, このフィールドは CallGenerator の GrowableArray を格納するフィールド.
この中に, その Compile オブジェクト内で生成された全ての LateInlineCallGenerator オブジェクトが格納されている)

(このフィールドは Compile::Compile( ciEnv* ci_env, C2Compiler* compiler, ciMethod* target, int osr_bci, bool subsume_loads, bool do_escape_analysis ) の終盤に参照されてインライン展開処理が行われる)

#### 生成箇所(where its instances are created)
CallGenerator::for_late_inline() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_call()
-> Compile::call_generator()
   -> CallGenerator::for_late_inline()
```




### 詳細(Details)
See: [here](../doxygen/classLateInlineCallGenerator.html) for details

---
## <a name="nolJZmNFWa" id="nolJZmNFWa">WarmCallGenerator</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: InlineWarmCalls).

CallGenerator クラスの具象サブクラスの1つ.
このクラスは, インライン展開するかどうかを後で判断したいメソッド呼び出し用
(WarmCallInfo による判定で hot でも cold でもない中間の温度だった場合にこのクラスが使用される).


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    //---------------------------WarmCallGenerator--------------------------------
    // Internal class which handles initial deferral of inlining decisions.
    class WarmCallGenerator : public CallGenerator {
```

### 使われ方(Usage)
CallGenerator::for_warm_call() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_call()
-> Compile::call_generator()
   -> CallGenerator::for_warm_call()
```




### 詳細(Details)
See: [here](../doxygen/classWarmCallGenerator.html) for details

---
## <a name="nohgmzQhGZ" id="nohgmzQhGZ">PredictedCallGenerator</a>

### 概要(Summary)
CallGenerator クラスの具象サブクラスの1つ.

このクラスは, ダイナミックディスパッチ(receiver の型チェック)が必要なメソッド呼び出しの最適化用
(e.g. invokevirtual, invokeinterface).

ダイナミックディスパッチが必要な場合でも, 
ある特定の型がレシーバーになる可能性が非常に高い場合には, それに特化した呼び出しにすることで高速化できる
(実行時に型をチェックし, あっていれば型に特化した呼び出し(可能ならさらにインライン展開も行う), 
違っていれば普通のダイナミックディスパッチにする, 等).

(なお, 初回呼び出し時のレシーバーを記憶する Inline Caching とは異なり, 
こちらはインタープリタ実行時のプロファイル情報を使用する.
プロファイル情報は type profile (ReceiverTypeDataオブジェクト) 内に格納されている).

(ところでここに書かれているコメントは VirtualCallGenerator クラスのものでは?? #TODO)


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    //------------------------PredictedCallGenerator------------------------------
    // Internal class which handles all out-of-line calls checking receiver type.
    class PredictedCallGenerator : public CallGenerator {
```

### 使われ方(Usage)
CallGenerator::for_predicted_call() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_call()
-> Compile::call_generator()
   -> CallGenerator::for_predicted_call()
```

### 内部構造(Internal structure)
コンストラクタで 2種類の CallGenerator オブジェクト (if_hit, if_missed) を受け取る.
これらがそれぞれ, 型が合っていた場合の呼び出し処理, 合っていなかった場合の呼び出し処理, を表す.
PredictedCallGenerator が生成するコードは, type check 結果に応じてどちらかを実行する.


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
      PredictedCallGenerator(ciKlass* predicted_receiver,
                             CallGenerator* if_missed,
                             CallGenerator* if_hit, float hit_prob)
```




### 詳細(Details)
See: [here](../doxygen/classPredictedCallGenerator.html) for details

---
## <a name="noU1PKDurZ" id="noU1PKDurZ">PredictedDynamicCallGenerator</a>

### 概要(Summary)
CallGenerator クラスの具象サブクラスの1つ.

このクラスは, java.lang.invoke.MethodHandle.invoke() の呼び出しの最適化用
(なお, invokedynamic による呼び出しとそうではない呼び出しの双方で使用されている).

ある特定の java.lang.invoke.MethodHandle オブジェクトがレシーバーになる可能性が非常に高い場合には, 
それに特化した呼び出しにすることで高速化できる
(実行時にオブジェクトをチェックし, あっていれば予定していたメソッドの呼び出し(可能ならさらにインライン展開も行う), 
違っていれば普通の呼び出しシーケンスにフォールバックする, 等).

(ところでここに書かれているコメントは VirtualCallGenerator クラスのものでは?? #TODO)


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    //------------------------PredictedDynamicCallGenerator-----------------------
    // Internal class which handles all out-of-line calls checking receiver type.
    class PredictedDynamicCallGenerator : public CallGenerator {
```

### 使われ方(Usage)
以下の箇所で(のみ)生成されている.

* CallGenerator::for_predicted_dynamic_call()
* CallGenerator::for_method_handle_inline()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_call()
-> Compile::call_generator()
   -> CallGenerator::for_method_handle_inline()
   -> CallGenerator::for_predicted_dynamic_call()
```

### 内部構造(Internal structure)
コンストラクタで 2種類の CallGenerator オブジェクト (if_hit, if_missed) を受け取る.
これらがそれぞれ, MethodHandle オブジェクトが一致した場合の呼び出し処理, 一致しなかった場合の呼び出し処理, を表す.
PredictedDynamicCallGenerator が生成するコードは, チェック結果に応じてどちらかを実行する.


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
      PredictedDynamicCallGenerator(ciMethodHandle* predicted_method_handle,
                                    CallGenerator* if_missed,
                                    CallGenerator* if_hit,
                                    float hit_prob)
```




### 詳細(Details)
See: [here](../doxygen/classPredictedDynamicCallGenerator.html) for details

---
## <a name="noqIvz7O-g" id="noqIvz7O-g">UncommonTrapCallGenerator</a>

### 概要(Summary)
CallGenerator クラスの具象サブクラスの1つ.

このクラスは, PredictedCallGenerator と併用されるクラス.
Uncommon Trap 処理を表す Ideal を生成する
(PredictedCallGenerator で予定していた型が外れた場合に, 
Uncommon Trap を起こして JIT コンパイルのやり直しを促すために使用される)

(ところでここに書かれているコメントは VirtualCallGenerator クラスのものでは?? #TODO)


```
    ((cite: hotspot/src/share/vm/opto/callGenerator.cpp))
    //-------------------------UncommonTrapCallGenerator-----------------------------
    // Internal class which handles all out-of-line calls checking receiver type.
    class UncommonTrapCallGenerator : public CallGenerator {
```

### 使われ方(Usage)
CallGenerator::for_uncommon_trap() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
Parse::do_call()
-> Compile::call_generator()
   -> CallGenerator::for_uncommon_trap()
```




### 詳細(Details)
See: [here](../doxygen/classUncommonTrapCallGenerator.html) for details

---
