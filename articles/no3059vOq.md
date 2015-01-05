---
layout: default
title: Thread の終了待ち処理 (java.lang.Thread.join() の処理)  
---
[Up](no1IkYYOWe.html) [Top](../index.html)

#### Thread の終了待ち処理 (java.lang.Thread.join() の処理)  

--- 
## 概要(Summary)
java.lang.Thread.join() は全て Java レベルのコードで実装されている.

処理としては, join 対象の Thread オブジェクトに対して java.lang.Object.wait() で待つだけ

## 備考(Notes)
java.lang.Object.wait() したスレッドを起こす処理は, 
JavaThread::exit() の処理における ensure_join() 内で行われる (See: [here](no2935w3j.html) for details).


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
java.lang.Thread.join()
-&gt; java.lang.Thread.join(long millis)
   -&gt; java.lang.Object.wait()
      (join 対象の Thread オブジェクトに対して java.lang.Object.wait() し, 終了するまで待つ.
       timeup 時間が指定されている場合は, 最大でその時間だけ待つ.
       スレッドが終了すると notify されるので wait が解ける.
       (See: <a href="no2935w3j.html">here</a> for details))
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.Thread.join(long millis, int nanos)
See: [here](no2114kZj.html) for details
### java.lang.Thread.join()
See: [here](no2114XPd.html) for details
### java.lang.Thread.join(long millis)
See: [here](no2114xjp.html) for details






