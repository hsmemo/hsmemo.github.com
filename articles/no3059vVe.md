---
layout: default
title: Exception の処理 ： その他 ： uncaughtException の処理  
---
[Up](noZknaL7f-.html) [Top](../index.html)

#### Exception の処理 ： その他 ： uncaughtException の処理  

--- 
## 概要(Summary)
uncaughtException の処理は, スレッドが終了する際の JavaThread::exit() の中で実行される (See: [here](no2935w3j.html) for details).

(JDK 1.5 以降の場合には) java.lang.Thread.dispatchUncaughtException() が呼び出され, 以下の優先度で処理される.

  1. java.lang.Thread.setUncaughtExceptionHandler() で UncaughtExceptionHandler がセットされていた場合:

     セットされている UncaughtExceptionHandler が呼び出される.

  2. 対象スレッドが, java.lang.ThreadGroup.uncaughtException() をオーバーライドした ThreadGroup に登録されていた場合:

     その java.lang.ThreadGroup.uncaughtException() メソッドが呼び出される.

     (なお, 登録されている ThreadGroup に親がいる場合は再帰的に調べる)

  3. java.lang.Thread.setDefaultUncaughtExceptionHandler() でデフォルトの UncaughtExceptionHandler がセットされていた場合:

     デフォルトの UncaughtExceptionHandler が呼び出される.

  4. 上記以外の場合:

     適当なエラーメッセージとスタックトレースを出力するのみ.

## 備考(Notes)
JDK 1.4 以前の場合には java.lang.ThreadGroup.uncaughtException() が呼び出される.


## 処理の流れ (概要)(Execution Flows : Summary)
### uncaughtException ハンドリング  (JDK 1.5 以降)
<div class="flow-abst"><pre>
(See: <a href="no2935w3j.html">here</a> for details)
-&gt; JavaThread::exit()
   -&gt; JavaCalls::call_virtual()
      -&gt; (See: <a href="no3059iJu.html">here</a> for details)
         -&gt; java.lang.Thread.dispatchUncaughtException()
            -&gt; java.lang.Thread.getUncaughtExceptionHandler()
               -&gt; * UncaughtExceptionHandler がセットされていれば, それが返される.
                  * そうでなければ ThreadGroup が返される.
            -&gt; java.lang.Thread.UncaughtExceptionHandler.uncaughtException()
               -&gt; * UncaughtExceptionHandler がセットされている場合には,
                    このメソッドをオーバーライドしたクラスが登録されているはずなので,
                    ここでユーザー指定の挙動が行われる.
                  * UncaughtExceptionHandler をセットされておらず ThreadGroup が返された場合には,
                    java.lang.ThreadGroup.uncaughtException() にフォールバックする.
</pre></div>


### uncaughtException ハンドリング  (JDK 1.4 以前)
<div class="flow-abst"><pre>
(See: <a href="no2935w3j.html">here</a> for details)
-&gt; JavaThread::exit()
   -&gt; JavaCalls::call_virtual()
      -&gt; (See: <a href="no3059iJu.html">here</a> for details)
         -&gt; java.lang.ThreadGroup.uncaughtException()
            -&gt; * このメソッドをオーバーライドした ThreadGroup が登録されている場合には, それが呼び出される.
               * オーバーライドされていない場合, 以下のデフォルトの java.lang.ThreadGroup.uncaughtException() の挙動にフォールバック.
                 -&gt; java.lang.Thread.getDefaultUncaughtExceptionHandler()
                 -&gt; java.lang.Thread.UncaughtExceptionHandler.uncaughtException()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.Thread.setUncaughtExceptionHandler()
(#Under Construction)
See: [here](no30598Yw.html) for details
### java.lang.Thread.getUncaughtExceptionHandler()
See: [here](no3059Jj2.html) for details
### java.lang.Thread.dispatchUncaughtException()
See: [here](no30597sF.html) for details
### java.lang.Thread.setDefaultUncaughtExceptionHandler()
(#Under Construction)
See: [here](no3059iLY.html) for details
### java.lang.Thread.getDefaultUncaughtExceptionHandler()
See: [here](no3059VBS.html) for details
### java.lang.ThreadGroup.uncaughtException()
See: [here](no3059I3L.html) for details






