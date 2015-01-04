---
layout: default
title: Class のロード/リンク/初期化 ： 初期化処理 (2) ： 初期化処理の流れ
---
[Up](no6dqMzJWt.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： 初期化処理 (2) ： 初期化処理の流れ

--- 
## 概要(Summary)
クラスの初期化処理は Klass::initialize() で行われる.
なお, このメソッドは各サブクラス(instanceKlass, objArrayKlass, typeArrayKlass) でオーバーライドされており,
実際にはサブクラスのメソッドが呼び出される.

初期化処理では, 以下の処理が行われる.

  * リンク処理の実行 (まだリンクされていない場合) (See: [here](no3059xqe.html) for details)

  * スーパークラスの初期化処理の実行 (まだスーパークラスが初期化されていない場合)

  * static 初期化子 (= <clinit> メソッド) の実行


## 処理の流れ (概要)(Execution Flows : Summary)
```
Klass::initialize() (<= 実際にはこのメソッドの中身は必要なサブクラスでオーバーライドされている. オーバーライドされていないサブクラスについては呼ばれることはない)
-> * instanceKlass の場合:
     -> instanceKlass::initialize()
        -> instanceKlass::initialize_impl()
           -> (1) (初期化前にリンクされていないとまずいので) リンクが終わった状態にしておく
                  -> instanceKlass::link_class()
                     -> (See: [here](no3059xqe.html) for details)
              (1) 他のスレッドが同時に初期化しようとしている可能性があるため, 排他を取る.
                  (他のスレッドが初期化中であればここで待機.
                   また, この時点で初期化されていたりあるいはエラーになっていれば, ここで終了)
                  (Java 仮想マシン仕様の "5.5 Initialization" における Step 1. ~ Step 6 の内容)
                  -> ObjectLocker::waitUninterruptibly() (※1)
                  -> instanceKlass::set_init_state()
                  -> instanceKlass::set_init_thread()
              (1) スーパークラスの初期化が必要であれば, 先にスーパークラスを初期化する.
                  (Java 仮想マシン仕様の "5.5 Initialization" における Step 7. の内容)
                  -> instanceKlass::initialize()  (super に対して再帰呼び出し)
              (1) 対象クラスを初期化する (= <clinit> メソッドを呼び出す)
                  (Java 仮想マシン仕様の "5.5 Initialization" における Step 9. の内容)
                  -> instanceKlass::call_class_initializer()
                     -> instanceKlass::call_class_initializer_impl()
                        -> instanceKlass::class_initializer()
                        -> JavaCalls::call()
                           -> (See: [here](no3059iJu.html) for details)
              (1) もし初期化処理が正常に終了した場合は, 排他を解除して終了
                  (Java 仮想マシン仕様の "5.5 Initialization" における Step 10. の内容)
                  -> instanceKlass::set_initialization_state_and_notify()
                     -> instanceKlass::set_initialization_state_and_notify_impl()
                        -> instanceKlass::set_init_state()
                        -> ObjectLocker::notifyAll() (※1で待機しているスレッドを起こす)
              (1) もし初期化処理で例外が出ていれば, 排他を解除した後, 例外を投げ直す
                  (Java 仮想マシン仕様の "5.5 Initialization" における Step 11. 及び Step 12. の内容)
                  -> instanceKlass::set_initialization_state_and_notify()
                     -> (同上)
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
See: [here](no75175VP.html) for details
### instanceKlass::should_be_initialized()
See: [here](no75175cD.html) for details
### instanceKlass::initialize_impl()
See: [here](no7517uUL.html) for details
### instanceKlass::set_init_state()
See: [here](no7517Exu.html) for details
(#Note #ifdef ASSERT 時には少し違うバージョンが定義されている)

### instanceKlass::set_init_thread()
See: [here](no7517Fy1.html) for details
### instanceKlass::call_class_initializer()
See: [here](no7517UoC.html) for details
### instanceKlass::call_class_initializer_impl()
See: [here](no7517uDD.html) for details
### instanceKlass::class_initializer()
See: [here](no7517ww0.html) for details
### instanceKlass::set_initialization_state_and_notify()
See: [here](no7517jMT.html) for details
### instanceKlass::set_initialization_state_and_notify_impl()
See: [here](no7517xQs.html) for details
### objArrayKlass::initialize()
See: [here](no7954Zvx.html) for details
### typeArrayKlass::initialize()
See: [here](no7954-Sq.html) for details






