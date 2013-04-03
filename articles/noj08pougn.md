---
layout: default
title: HotSpot の起動/終了処理(Starting/Quitting)
---
[Up](no1S0Auo49.html) [Top](../index.html)

#### HotSpot の起動/終了処理(Starting/Quitting)

--- 
## 概要(Summary)
(HotSpot は大抵の場合 "java" コマンド経由で起動されるので, 以下ではその場合について説明する)

java コマンドによる HotSpot の起動/終了処理 (= Java プログラムの実行処理) は, 
以下の 3つのコンポーネントによって実現されている.

なお最終的には, 
JNI の Invocation API である JNI_CreateJavaVM()/DestroyJavaVM() と
JNI 関数である CallStaticVoidMethod() によって Java プログラムの実行が実現される
(JNI_CreateJavaVM() によって HotSpot を生成&初期化し, 
CallStaticVoidMethod() によって main() メソッドを呼び出す.
main() が終わったら DestroyJavaVM() で HotSpot を破棄する).

  * java コマンド

    ユーザーが使用するコマンド (launcher). 
    
    内部的には, 単に libjli を呼び出しているだけ.

  * libjli ライブラリ
    
    launcher (java コマンド等) から呼ばれるライブラリ.

    HotSpot を起動させるための関数 (JLI_Launch() 等) が定義されている.

    これが実質的な launcher の本体で, ここから libjvm が呼び出される.
    なお, jli は "Java Launcher Infrastructure" の略とのこと.

  * libjvm ライブラリ
    
    実際の HotSpot としての機能を提供しているライブラリ.
    
    なお別の言い方をすると, これは JNI の機能を提供するライブラリ. 
    この中に JNI の Invocation API や JNI 関数が定義されている.




## Subcategories
* [HotSpot の起動/終了処理 ： java コマンド (launcher) 部分の処理の流れ](nokAq80V3Y.html)
* [HotSpot の起動/終了処理 ： HotSpot の起動処理の流れ (JNI_CreateJavaVM() の処理の流れ)   ](no2114J7x.html)
* [HotSpot の起動/終了処理 ： HotSpot の終了処理の流れ ](no28916GoL.html)



