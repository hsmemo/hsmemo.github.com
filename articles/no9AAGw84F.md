---
layout: default
title: Class のロード/リンク/初期化 ： 初期化処理(Initializing)
---
[Up](no7882ALm.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： 初期化処理(Initializing)

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
```
Klass::initialize() (<= 実際にはこのメソッドの中身は必要なサブクラスでオーバーライドされている. オーバーライドされていないサブクラスについては呼ばれることはない)
-> * instanceKlass の場合:
     -> instanceKlass::initialize()
        -> instanceKlass::initialize_impl()
           -> (1) (初期化前にリンクされていないとまずいので) リンクが終わった状態にしておく
                  -> instanceKlass::link_class()
                     -> (See: [here](no3059xqe.html) for details)
              (1) Step 1. ~ Step 6 他のスレッドが同時に初期化しようとしている可能性があるため, 排他を取る.
                  (この時点で初期化されていたりあるいはエラーになっていれば, ここで終了)
                  -> instanceKlass::set_init_state()
                  -> instanceKlass::set_init_thread()
              (1) Step 7. スーパークラスの初期化が必要であれば, 先にスーパークラスを初期化する.
                  -> instanceKlass::initialize()
              (1) Step 8. 対象クラスを初期化する (= <clinit> メソッドを呼び出す)
                  -> instanceKlass::call_class_initializer()
                     -> instanceKlass::call_class_initializer_impl()
                        -> instanceKlass::class_initializer()
                        -> JavaCalls::call()
                           -> (See: [here](no3059iJu.html) for details)
              (1) Step 9. もし初期化処理で例外が出ていなければ, 排他を解除して終了
                  -> instanceKlass::set_initialization_state_and_notify()
              (1) Step 10. ~ Step 11. もし例外が出ていれば, 排他を解除した後, 例外を投げ直す
                  -> instanceKlass::set_initialization_state_and_notify()
                  -> THROW_OOP() or THROW_ARG()

   * objArrayKlass の場合:
     -> objArrayKlass::initialize()
        -> instanceKlass::initialize() or typeArrayKlass::initialize()

   * typeArrayKlass の場合:
     -> typeArrayKlass::initialize()
        -> (何もしない)

   * その他の場合(オーバーライドされていない場合)
     -> Klass::initialize()
        -> ShouldNotReachHere()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### Klass::initialize()
See: [here](no7954-L2.html) for details
### instanceKlass::initialize()
(#Under Construction)

### objArrayKlass::initialize()
See: [here](no7954Zvx.html) for details
### typeArrayKlass::initialize()
See: [here](no7954-Sq.html) for details






