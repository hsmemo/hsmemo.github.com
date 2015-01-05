---
layout: default
title: RFrame クラス関連のクラス (RFrame, CompiledRFrame, InterpretedRFrame, DeoptimizedRFrame)
---
[Top](../index.html)

#### RFrame クラス関連のクラス (RFrame, CompiledRFrame, InterpretedRFrame, DeoptimizedRFrame)

これらは, C2 JIT Compiler 用の補助クラス.
より具体的に言うと, StackWalkCompPolicy クラスの処理で使用される一時オブジェクト (See: [here](no34200pY.html) for details).


### クラス一覧(class list)

  * [RFrame](#no49jMcPts)
  * [CompiledRFrame](#noYZ4n4Cmd)
  * [InterpretedRFrame](#no6oJG9vWV)
  * [DeoptimizedRFrame](#no8F9_lWCC)


---
## <a name="no49jMcPts" id="no49jMcPts">RFrame</a>

### 概要(Summary)
StackWalkCompPolicy クラス内で使用される補助クラス(の基底クラス).

StackWalkCompPolicy が (処理対象のメソッドを決めるために) 
スタックフレームを調べる処理で使用される一時オブジェクト(ResourceObjクラス).

1つの RFrame オブジェクトが 1つのスタックフレームに対応する.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/rframe.hpp))
    // rframes ("recompiler frames") decorate stack frames with some extra information
    // needed by the recompiler.  The recompiler views the stack (at the time of recompilation)
    // as a list of rframes.
    
    class RFrame : public ResourceObj {
```




### 詳細(Details)
See: [here](../doxygen/classRFrame.html) for details

---
## <a name="noYZ4n4Cmd" id="noYZ4n4Cmd">CompiledRFrame</a>

### 概要(Summary)
RFrame クラスの具象サブクラスの1つ 
(= StackWalkCompPolicy クラス内で使用される一時オブジェクト(ResourceObjクラス)).

このクラスは JIT コンパイルされたメソッドのスタックフレームを表す.

1つの CompiledRFrame オブジェクトが 1つのスタックフレームに対応する.
ただし, インライン展開された場合は実際のスタックフレームが Java レベルのメソッドと1対1対応しないことがある
(実際のスタックフレーム 1 に対して Java レベルのメソッド n 個が対応).
CompiledRFrame は Java レベルのメソッドと 1対1対応するような論理的なスタックフレームを表す
(つまり, 実際のスタックフレームとは1対1対応しないことがある).


```cpp
    ((cite: hotspot/src/share/vm/runtime/rframe.hpp))
    class CompiledRFrame : public RFrame {    // frame containing a compiled method
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
RFrame::new_RFrame() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

(ただし, ResourceObjクラスなので一時的なオブジェクト)

<div class="flow-abst"><pre>
(略) (See: <a href="no34200pY.html">here</a> for details)
-&gt; StackWalkCompPolicy::method_invocation_event()
   -&gt; StackWalkCompPolicy::findTopInlinableFrame()
      -&gt; StackWalkCompPolicy::senderOf()
         -&gt; RFrame::caller()
            -&gt; RFrame::new_RFrame()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classCompiledRFrame.html) for details

---
## <a name="no6oJG9vWV" id="no6oJG9vWV">InterpretedRFrame</a>

### 概要(Summary)
RFrame クラスの具象サブクラスの1つ
(= StackWalkCompPolicy クラス内で使用される一時オブジェクト(ResourceObjクラス)).

このクラスはインタープリタで実行されているメソッドのスタックフレームを表す.

1つの InterpretedRFrame オブジェクトが 1つのスタックフレームに対応する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/rframe.hpp))
    class InterpretedRFrame : public RFrame {    // interpreter frame
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている (See: [here](no34200pY.html) for details).

(ただし, ResourceObjクラスなので一時的なオブジェクト)

* RFrame::new_RFrame()
* StackWalkCompPolicy::method_invocation_event()

なお, RFrame::new_RFrame() は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
(略) (See: <a href="no34200pY.html">here</a> for details)
-&gt; StackWalkCompPolicy::method_invocation_event()
   -&gt; StackWalkCompPolicy::findTopInlinableFrame()
      -&gt; StackWalkCompPolicy::senderOf()
         -&gt; RFrame::caller()
            -&gt; RFrame::new_RFrame()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classInterpretedRFrame.html) for details

---
## <a name="no8F9_lWCC" id="no8F9_lWCC">DeoptimizedRFrame</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```cpp
    ((cite: hotspot/src/share/vm/runtime/rframe.hpp))
    // treat deoptimized frames as interpreted
    class DeoptimizedRFrame : public InterpretedRFrame {
```




### 詳細(Details)
See: [here](../doxygen/classDeoptimizedRFrame.html) for details

---
