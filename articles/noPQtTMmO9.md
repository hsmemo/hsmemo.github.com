---
layout: default
title: JNI の処理 ： native method の処理 ： native method の呼び出し処理 ： JIT コンパイル対象になっていない native method の場合 ： Template Interpreter の場合
---
[Up](no3059asZ.html) [Top](../index.html)

#### JNI の処理 ： native method の処理 ： native method の呼び出し処理 ： JIT コンパイル対象になっていない native method の場合 ： Template Interpreter の場合

--- 
## 概要(Summary)
(#Under Construction)

## 備考(Notes)
ネイティブメソッドの呼び出し時には, 引数や返値をネイティブの ABI に合わせて変換する関数が用いられる.
これらはそれぞれ「シグネチャハンドラ(signature handler)」と「リザルトハンドラ(result handler)」と呼ばれる.

* シグネチャハンドラ(signature handler)
  
  オペランドスタックの引数を ABI にしたがった形に変換する関数

* リザルトハンドラ(result handler)
  
  ネイティブメソッドの返り値をオペランドスタックに載せる関数
  (Sparc 版ではついでにフレームの開放も行う)

なお, シグネチャハンドラ/リザルトハンドラ はペアで扱われ,
シグネチャハンドラ内で Lscratch レジスタ(Sparc の場合) や rax レジスタ(x86_64 の場合)にそのメソッドの返り値に対応するリザルトハンドラのアドレスが格納される.

また, シグネチャハンドラとリザルトハンドラはそれぞれ引数の型/返り値の型が共通の同士は流用できるので,
シグネチャ文字列 (= 型を表す文字列. 例: "Ljava/lang/Object") をキーとする hashmap に登録して管理している
(See: SignatureHandlerLibrary).

実際のデータ構造上では, 各 methodOop オブジェクト内の signature_handler フィールドに
「そのメソッドを呼ぶ際に使用する signature handler」 が格納されている.
InterpreterGenerator::generate_native_entry() が生成したコードは
ここから signature handler を取ってきて引数の詰め直しに使用する


## 備考(Notes)
* slow case では Interpreter::slow_signature_handler() が指しているコードが用いられる.
  このコードは初期化時に生成される.

* また, result handler のコードも初期化時に生成される.


## 処理の流れ (概要)(Execution Flows : Summary)
### 初期化時の処理 
```
* slow_signature_handler の生成処理

  (See: [here](no3059SwU.html) for details)
  -> TemplateInterpreterGenerator::generate_all()
     -> AbstractInterpreterGenerator::generate_all()
        -> AbstractInterpreterGenerator::generate_slow_signature_handler()

* result handler の生成処理

  (See: [here](no3059SwU.html) for details)
  -> TemplateInterpreterGenerator::generate_all()
     -> TemplateInterpreterGenerator::generate_result_handler_for()

* method entry 部コードの生成処理

  (See: [here](no3059n2f.html) for details)
  -> AbstractInterpreterGenerator::generate_method_entry()
     -> InterpreterGenerator::generate_native_entry()
```


### ネイティブメソッドの呼び出し処理 : sparc の場合
```
InterpreterGenerator::generate_native_entry() が生成するコード
-> (1) スタック上に新しいフレームを確保する
       -> TemplateInterpreterGenerator::generate_fixed_frame() が生成したコード

   (1) JIT コンパイラを使う場合は, invocation counter 値を増加させて値をチェックする
       -> InterpreterGenerator::generate_counter_incr() が生成したコード

   (1) stack overflow をチェックする (stack banging コード)
       -> AbstractInterpreterGenerator::bang_stack_shadow_pages() が生成したコード

   (1) synchronized method の場合はロックを取得する
       -> InterpreterGenerator::lock_method() が生成したコード
          -> (See: [here](no9662EYy.html) for details)

   (1) (まだ設定されていなければ) 対応するネイティブ関数のアドレスやシグネチャハンドラを設定しておく
       -> InterpreterRuntime::prepare_native_call()
          -> (1) (もしまだ終わっていなければ) 呼び出し先の関数へのダイナミックリンクを行う
                 -> NativeLookup::lookup()
                    -> (後述) (See: [here](no3059err.html) for details)
             (1) (もしまだ終わっていなければ) シグネチャハンドラの設定を行う
                 -> SignatureHandlerLibrary::add()
                    -> * 既に (1回以上呼び出されているため) 設定済みの場合: 
                         -> 何もしない
                       * 何らかの理由で特注のシグネチャハンドラが用意できないメソッドの場合 (引数の数が多すぎる, 等):
                         -> AbstractInterpreter::slow_signature_handler()  (<= デフォルトのハンドラが返される)
                       * 既に同じ型用のシグネチャハンドラを作ってメモイズしてある場合:
                         -> それを流用する
                       * それ以外の場合: 
                         -> InterpreterRuntime::SignatureHandlerGenerator::generate() (<= シグネチャハンドラを作成)
                            -> SignatureIterator::iterate()
                               -> SignatureIterator::parse_type()
                                  -> NativeSignatureIterator::do_<type>()
                                     -> InterpreterRuntime::SignatureHandlerGenerator::pass_*()
                                        -> MacroAssembler::store_*_argument()

   (1) ダミーフレームを作ってレジスタを待避する

   (1) シグネチャハンドラを呼んで引数を詰め直す

   (1) native メソッドの呼び出しを行う
       -> (native メソッドの処理が実行される)
    
   (1) もし Safepoint 停止が開始されていた場合は, ここで block する. 
       -> JavaThread::check_special_condition_for_native_trans()

   (1) スタック上の yellow page が disabled になっていたら, 元に戻す.
       -> SharedRuntime::reguard_yellow_pages()

   (1) 例外が起きていれば伝搬させる. この場合は以降の処理は行わない.
       -> MacroAssembler::call_VM()
          -> (See: [here](no2935dSX.html) for details)
             -> InterpreterRuntime::throw_pending_exception()
                -> (See: [here](no3059d4M.html) for details)

   (1) synchronized method の場合はロックを解放する
       -> InterpreterGenerator::unlock_method() が生成したコード
          -> (See: [here](no3059F5A.html) for details)

   (1) リザルトハンドラを実行する

   (1) ダミーフレームを削除する

   (1) 呼び出し元にリターン
```

### ネイティブメソッドの呼び出し処理 : x86_64 の場合
```
InterpreterGenerator::generate_native_entry() が生成するコード
-> (1) スタック上に新しいフレームを確保する
       -> TemplateInterpreterGenerator::generate_fixed_frame() が生成したコード

   (1) JIT コンパイラを使う場合は, invocation counter 値を増加させて値をチェックする
       -> InterpreterGenerator::generate_counter_incr() が生成したコード

   (1) stack overflow をチェックする (stack banging コード)
       -> AbstractInterpreterGenerator::bang_stack_shadow_pages() が生成したコード

   (1) synchronized method の場合はロックを取得する
       -> InterpreterGenerator::lock_method() が生成したコード
          -> (See: [here](no9662EYy.html) for details)

   (1) ダミーフレームを作る

   (1) (まだ設定されていなければ) 対応するネイティブ関数のアドレスやシグネチャハンドラを設定しておく
       -> InterpreterRuntime::prepare_native_call()
          -> (同上)

   (1) シグネチャハンドラを呼んで引数を詰め直す

   (1) native メソッドの呼び出しを行う
       -> (native メソッドの処理が実行される)
    
   (1) もし Safepoint 停止が開始されていた場合は, ここで block する. 
       -> JavaThread::check_special_condition_for_native_trans()

   (1) レジスタの復帰させ, ダミーフレームを削除する

   (1) 例外が起きていれば伝搬させる. この場合は以降の処理は行わない.
       -> MacroAssembler::call_VM()
          -> (See: [here](no2935dSX.html) for details)
             -> InterpreterRuntime::throw_pending_exception()
                -> (See: [here](no3059d4M.html) for details)

   (1) synchronized method の場合はロックを解放する
       -> InterpreterGenerator::unlock_method() が生成したコード
          -> (See: [here](no3059F5A.html) for details)

   (1) リザルトハンドラを実行する

   (1) 呼び出し元にリターン
```



## 処理の流れ (詳細)(Execution Flows : Details)
### AbstractInterpreterGenerator::generate_slow_signature_handler() (sparc 64bit の場合)
See: [here](no3059QuG.html) for details
### AbstractInterpreterGenerator::generate_slow_signature_handler() (sparc 32bit の場合)
See: [here](no1695Vjd.html) for details
### InterpreterRuntime::slow_signature_handler() (sparc の場合)
See: [here](no3059rDa.html) for details
### SlowSignatureHandler::pass_int()
(#Under Construction)
See: [here](no3059Tjz.html) for details
### SlowSignatureHandler::pass_long()
(#Under Construction)

### SlowSignatureHandler::pass_float()
(#Under Construction)

### SlowSignatureHandler::pass_double()
(#Under Construction)

### SlowSignatureHandler::pass_object()
(#Under Construction)


### AbstractInterpreterGenerator::generate_slow_signature_handler() (x86_64 && Windows の場合)
See: [here](no1695ILv.html) for details
### AbstractInterpreterGenerator::generate_slow_signature_handler() (x86_64 && 非Windows の場合)
See: [here](no1695hsc.html) for details
### InterpreterRuntime::slow_signature_handler() (x86_64 の場合)
See: [here](no3059FtC.html) for details


### TemplateInterpreterGenerator::generate_result_handler_for()  (sparc の場合)
See: [here](no3059FmO.html) for details
### TemplateInterpreterGenerator::generate_result_handler_for()  (x86_64 の場合)
See: [here](no3059f6a.html) for details
### MacroAssembler::c2bool()  (x86 の場合)
See: [here](no3059GZt.html) for details
### MacroAssembler::sign_extend_byte()  (x86 の場合)
See: [here](no3059sEh.html) for details
### MacroAssembler::sign_extend_short()  (x86 の場合)
See: [here](no30595On.html) for details


### InterpreterGenerator::generate_native_entry() (Sparc の場合)
(#Under Construction)
See: [here](no3718Gvm.html) for details
### InterpreterGenerator::generate_native_entry() (x86_64 の場合)
(#Under Construction)
See: [here](no1695P8Y.html) for details

### TemplateInterpreterGenerator::generate_fixed_frame()
(See: [here](no2935G1h.html) for details)

### InterpreterGenerator::generate_counter_incr()
(See: [here](no2935G1h.html) for details)

### AbstractInterpreterGenerator::bang_stack_shadow_pages()
(See: [here](no2935G1h.html) for details)


### InterpreterRuntime::prepare_native_call()
See: [here](no17119qgA.html) for details
### methodOopDesc::has_native_function()
See: [here](no30593MZ.html) for details
### methodOopDesc::native_function()
See: [here](no3059EXf.html) for details
### methodOopDesc::native_function_addr()
See: [here](no3059Rhl.html) for details
### methodOopDesc::signature_handler()
See: [here](no3059d_A.html) for details
### methodOopDesc::signature_handler_addr()
See: [here](no30593TN.html) for details
### methodOopDesc::set_signature_handler()
See: [here](no3059FRy.html) for details

### SignatureHandlerLibrary::add()
See: [here](no3059r1x.html) for details
### SignatureHandlerLibrary::initialize()
See: [here](no3059EeT.html) for details
### SignatureHandlerLibrary::set_handler()
See: [here](no3059RoZ.html) for details
### SignatureHandlerLibrary::set_handler_blob()
See: [here](no3059eyf.html) for details
### SignatureHandlerLibrary::pd_set_handler() (sparc の場合)
See: [here](no3059r8l.html) for details
### SignatureHandlerLibrary::pd_set_handler() (x86_64 の場合)
See: [here](no30594Gs.html) for details

### InterpreterRuntime::SignatureHandlerGenerator::generate()  (sparc の場合)
See: [here](no30593aB.html) for details
### SignatureIterator::iterate()
See: [here](no30594Ng.html) for details
### SignatureIterator::expect()
See: [here](no3059Sis.html) for details
### SignatureIterator::parse_type()
See: [here](no3059FYm.html) for details
### SignatureIterator::check_signature_end()
See: [here](no3059fsy.html) for details
### NativeSignatureIterator::do_<type>()  (NativeSignatureIterator::do_bool(), NativeSignatureIterator::do_char(), NativeSignatureIterator::do_float(), NativeSignatureIterator::do_double(), ...)

```
    ((cite: hotspot/src/share/vm/runtime/signature.hpp))
      void do_bool  ()                     { pass_int();    _jni_offset++; _offset++;       }
      void do_char  ()                     { pass_int();    _jni_offset++; _offset++;       }
      void do_float ()                     { pass_float();  _jni_offset++; _offset++;       }
    #ifdef _LP64
      void do_double()                     { pass_double(); _jni_offset++; _offset += 2;    }
    #else
      void do_double()                     { pass_double(); _jni_offset += 2; _offset += 2; }
    #endif
      void do_byte  ()                     { pass_int();    _jni_offset++; _offset++;       }
      void do_short ()                     { pass_int();    _jni_offset++; _offset++;       }
      void do_int   ()                     { pass_int();    _jni_offset++; _offset++;       }
    #ifdef _LP64
      void do_long  ()                     { pass_long();   _jni_offset++; _offset += 2;    }
    #else
      void do_long  ()                     { pass_long();   _jni_offset += 2; _offset += 2; }
    #endif
      void do_void  ()                     { ShouldNotReachHere();                               }
      void do_object(int begin, int end)   { pass_object(); _jni_offset++; _offset++;        }
      void do_array (int begin, int end)   { pass_object(); _jni_offset++; _offset++;        }
```

### InterpreterRuntime::SignatureHandlerGenerator::pass_int()  (sparc の場合)
See: [here](no3059eAI.html) for details
### InterpreterRuntime::SignatureHandlerGenerator::pass_word()  (sparc の場合)
See: [here](no3059rKO.html) for details
### InterpreterRuntime::SignatureHandlerGenerator::pass_long()  (sparc の場合)
See: [here](no3059Ffa.html) for details
### InterpreterRuntime::SignatureHandlerGenerator::pass_float()  (sparc の場合)
See: [here](no3059Spg.html) for details
### InterpreterRuntime::SignatureHandlerGenerator::pass_double()  (sparc の場合)
See: [here](no3059fzm.html) for details
### InterpreterRuntime::SignatureHandlerGenerator::pass_object()  (sparc の場合)
See: [here](no3059s9s.html) for details
### MacroAssembler::store_argument()  (sparc の場合)
See: [here](no30594UU.html) for details
### MacroAssembler::store_float_argument()  (sparc 64bit の場合)
See: [here](no30595Hz.html) for details
### MacroAssembler::store_double_argument()  (sparc 64bit の場合)
See: [here](no3059rRC.html) for details
### MacroAssembler::store_ptr_argument()  (sparc の場合)
See: [here](no30594bI.html) for details

### InterpreterRuntime::SignatureHandlerGenerator::generate()  (x86_64 の場合)
(#Under Construction)
See: [here](no3059ElH.html) for details

### JavaThread::check_special_condition_for_native_trans()
(#Under Construction)

### InterpreterGenerator::save_native_result()
(#Under Construction)

### InterpreterGenerator::restore_native_result()
(#Under Construction)


### AbstractInterpreter::slow_signature_handler()
See: [here](no3059RvN.html) for details






