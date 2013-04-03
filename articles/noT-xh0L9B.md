---
layout: default
title: Exception の処理 ： 処理の詳細 (4) ： 例外ハンドラの探索処理 ： 同種のフレーム内での処理 ： ランタイムフレームでの処理
---
[Up](noAJsAY6Zl.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (4) ： 例外ハンドラの探索処理 ： 同種のフレーム内での処理 ： ランタイムフレームでの処理

--- 
## 概要(Summary)
(概要は別項参照 (See: [here](no3059qHd.html) for details))

HotSpot のランタイム内には基本的には例外を捕捉する箇所はない.
このため, 一度ランタイム内に例外が入るとランタイムのフレームを全て突き抜けるまで unwind が繰り返される.
この処理はランタイム内で明示的に pending_exception の確認とリターンを繰り返すことで実現されている.

この処理の記述を容易化するため, マクロが用意されている.
pending_exception をセットしうる関数を呼ぶ箇所では
(特に後始末などが不要で例外が起きていた場合は即座に return していいケースでは)
CHECK というマクロで例外を伝搬させていくのが慣習になっている.

### CHECK マクロの使用方法
pending_exception をセットしうる関数については,
関数定義において最終引数を Thread* THREAD にするのが慣習になっている
(あるいは, 最後の引数に TRAPS と書いておいてもよい.
 これは Thread* THREAD と #define されているので単なるシンタックスシュガー).

これらの関数を呼び出す際には, この最終引数に対応する位置に CHECK と書く.
関数内で例外が起きていれば, CHECK マクロはその関数からの return 直後にその場で即 return する. これで例外による伝搬の挙動がエミュレートされる.

* 使用例: SystemDictionary::resolve_or_fail() の場合:


```
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.cpp))
    klassOop SystemDictionary::resolve_or_fail(Symbol* class_name,
                                               bool throw_error, TRAPS)
    {
    ...
```


```
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
        klassOop access_controller_klass = SystemDictionary::resolve_or_fail(access_controller, false, CHECK);
```

## 備考(Notes)
なお, 例外が起こっていれば ShouldNotReachHere() で即座に異常終了させる CATCH というマクロもある.


## 詳細(Details)
以下のようなマクロが用意されている.

  * 例外を送出する可能性がある関数用のマクロ:

```
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    // The THREAD & TRAPS macros facilitate the declaration of functions that throw exceptions.
    // Convention: Use the TRAPS macro as the last argument of such a function; e.g.:
    //
    // int this_function_may_trap(int x, float y, TRAPS)
    
    #define THREAD __the_thread__
    #define TRAPS  Thread* THREAD
```

  * pending exception のチェック用のマクロ:

```
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    // The CHECK... macros should be used to pass along a THREAD reference and to check for pending
    // exceptions. In special situations it is necessary to handle pending exceptions explicitly,
    // in these cases the PENDING_EXCEPTION helper macros should be used.
    //
    // Macro naming conventions: Macros that end with _ require a result value to be returned. They
    // are for functions with non-void result type. The result value is usually ignored because of
    // the exception and is only needed for syntactic correctness. The _0 ending is a shortcut for
    // _(0) since this is a frequent case. Example:
    //
    // int result = this_function_may_trap(x_arg, y_arg, CHECK_0);
    //
    // CAUTION: make sure that the function call using a CHECK macro is not the only statement of a
    // conditional branch w/o enclosing {} braces, since the CHECK macros expand into several state-
    // ments!
    
    #define PENDING_EXCEPTION                        (((ThreadShadow*)THREAD)->pending_exception())
    #define HAS_PENDING_EXCEPTION                    (((ThreadShadow*)THREAD)->has_pending_exception())
    #define CLEAR_PENDING_EXCEPTION                  (((ThreadShadow*)THREAD)->clear_pending_exception())
    
    #define CHECK                                    THREAD); if (HAS_PENDING_EXCEPTION) return       ; (0
    #define CHECK_(result)                           THREAD); if (HAS_PENDING_EXCEPTION) return result; (0
    #define CHECK_0                                  CHECK_(0)
    #define CHECK_NH                                 CHECK_(Handle())
    #define CHECK_NULL                               CHECK_(NULL)
    #define CHECK_false                              CHECK_(false)
```

  * 例外を送出する関数を呼ぶ側で, 例外が出なかったことを保証しておくためのマクロ : (もしも例外が出ていたら即座に異常終了)

```
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    // The CATCH macro checks that no exception has been thrown by a function; it is used at
    // call sites about which is statically known that the callee cannot throw an exception
    // even though it is declared with TRAPS.
    
    #define CATCH                              \
      THREAD); if (HAS_PENDING_EXCEPTION) {    \
        oop ex = PENDING_EXCEPTION;            \
        CLEAR_PENDING_EXCEPTION;               \
        ex->print();                           \
        ShouldNotReachHere();                  \
      } (0
```


## 処理の流れ (概要)(Execution Flows : Summary)
(#Under Construction)

(CHECK の挙動は, 上記のマクロ定義から明らかな通り, 単にチェックしてリターンするだけ)

## 処理の流れ (詳細)(Execution Flows : Details)
(#Under Construction)

(CHECK の挙動は, 上記のマクロ定義から明らかな通り, 単にチェックしてリターンするだけ)







