---
layout: default
title: Class のロード/リンク/初期化 ： リンク処理 (3) ： バイトコードの検証処理(verification)  
---
[Up](noX5hsnWQw.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： リンク処理 (3) ： バイトコードの検証処理(verification)  

--- 
#Under Construction

## 概要(Summary)
(以下の内容は大部分が [HotSpot Runtime Overview : Bytecode Verifier and Format Checker](http://openjdk.java.net/groups/hotspot/docs/RuntimeOverview.html#Bytecode%20Verifier%20and%20Format%20Checker|outline) に基づく. こちらも参照のこと)

クラスのリンク中の verification 処理は Verifier::verify() で行われる.

なお, 現在の HotSpot 内には 2種類の verifier が実装されている.

  * Type Inference Verifier
    
    JDK 5 以前からあった verifier.
    
    抽象実行を用いて型推論を行っていき, 型情報の単一化を繰り返すことで検証を行う.
    検証は型情報が収束する(= 変化しなくなる)か, エラーが見つかれば終了する.
    
    実装は libverify.so 内に入っており, HotSpot 内のデータには JNI を用いてアクセスする
    (ソースコードは jdk/src/share/native/common/ 以下にある模様).
    
  * Type Checking Verifier (なお, 上記のページ中では "type verification" と呼称)
    
    Java 仮想マシン仕様バージョン 6 で追加された方式に基づく verifier.
    
    検証を容易化するため, あらかじめ Java コンパイラが型状態をクラスファイル中に埋め込んでおき, 
    検証処理はその情報を確認することで行う.
    (なお, 型情報はクラスファイル中の StackMapTable attribute として埋め込まれる).
    検証処理は, Type Inference のように収束するまで待つ必要はなく, ワンパスで完了する.
    
    非常に簡単で高速であるため, 検証処理は HotSpot 内部に実装されている (ClassVerifier クラス(?)).

クラスファイルのバージョンナンバーが 50 未満 (JDK 5 以下) の場合, type inference で検証する.
バージョンが 50 以上のクラスファイルであれば type checking で検証するが, 
StackMapTable attribute を変更せずにコード部分だけ変更する古いツールがあるかもしれないため, 
失敗した場合には type inference にフォールバックする.

## 参考(for your information)
* [HotSpot Runtime Overview : Bytecode Verifier and Format Checker](http://openjdk.java.net/groups/hotspot/docs/RuntimeOverview.html#Bytecode%20Verifier%20and%20Format%20Checker|outline)

* Java 仮想マシン仕様
  (4.10 "Verification of class Files", 
  4.10.1 "Verification by Type Checking",
  4.10.2 "Verification by Type Inference", 
  5.4.1 "Verification")

## 備考(Notes)
以下のコマンドラインオプションが verification 処理に影響する.

*  -Xverify
   
   verification 対象のクラスを制御する
   (なお, デフォルトではブートストラップ・クラスローダがロードしたクラスは verify 対象外).
   
   * -Xverify:all   -- ブートストラップ・クラスローダがロードしたクラスも verify する
   * -Xverify:none  -- 全てのクラスを verify 対象外とする

   なお, この情報は HotSpot 内部では以下の二つのフラグで扱われている
   (BytecodeVerificationLocal, BytecodeVerificationRemote)
   (See: Verifier::should_verify_for())

* UseSplitVerifier
  
  Type Checking verifier を使用するかどうかを設定
  (オプションをオフにした場合は新しいクラスファイルに対しても Type Inference verifier を使用).

* FailOverToOldVerifier

  Type Checking verifier が失敗した際に Type Inference verifier にフォールバックするかどうかを設定.

* ...

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
Verifier::verify()
-&gt; * type checking verifier を使用する場合
     -&gt; ClassVerifier::verify_class()
        -&gt; ClassVerifier::verify_method()
           -&gt; 

   * type inference verifier を使用する場合
     -&gt; Verifier::inference_verify()
        -&gt; (1) verify 処理用の関数を取得する
               (可能なら VerifyClassCodesForMajorVersion() を取得. だめなら VerifyClassCodes())
               -&gt; verify_byte_codes_fn()
                  -&gt; os::native_java_library()

           (1) 取得した verify 処理用の関数を呼び出す
               -&gt; * VerifyClassCodesForMajorVersion() の場合
                    -&gt; VerifyClassForMajorVersion()
                       -&gt; verify_field()
                       -&gt; verify_method()
                          -&gt; verify_opcode_operands()
                          -&gt; initialize_exception_table()
                          -&gt; initialize_dataflow()
                          -&gt; run_dataflow()
                          -&gt; verify_constant_pool_type()

                  * VerifyClassCodes() の場合
                    -&gt; VerifyClass()
                       -&gt; VerifyClassForMajorVersion()
                          -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### Verifier::verify()
See: [here](no18536MAS.html) for details
### Verifier::is_eligible_for_verification()
(#Under Construction)
See: [here](no18536eFn.html) for details
### Verifier::should_verify_for()
(#Under Construction)
See: [here](no2099kuD.html) for details
### ClassVerifier::verify_class()
See: [here](no18536deG.html) for details
### ClassVerifier::was_recursively_verified()
See: [here](no18536HHi.html) for details
### ClassVerifier::verify_method()
(#Under Construction)


### Verifier::inference_verify()
See: [here](no18536FFU.html) for details
### verify_byte_codes_fn()
See: [here](no18536Xil.html) for details
### VerifyClassCodesForMajorVersion()
See: [here](no18536rdV.html) for details
### VerifyClassCodes()
See: [here](no18536S8n.html) for details
### VerifyClassForMajorVersion()
(#Under Construction)
See: [here](no18536I12.html) for details
### VerifyClass()
See: [here](no18536tYv.html) for details






