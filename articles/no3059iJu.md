---
layout: default
title: Method に関する処理 ： ランタイム(Runtime)によるメソッド呼び出し処理 (JavaCalls：：call_*()) 
---
[Up](nosOtIT-Vy.html) [Top](../index.html)

#### Method に関する処理 ： ランタイム(Runtime)によるメソッド呼び出し処理 (JavaCalls：：call_*()) 

--- 
## 概要(Summary)
ランタイムによる Java メソッドの呼び出し処理は JavaCalls クラスに実装されている.
より具体的に言うと, JavaCalls クラスの以下の public メソッドが呼び出し処理になっている.

  * JavaCalls::call()
  * JavaCalls::call_default_constructor()
  * JavaCalls::call_virtual()            (<= 引数の異なる同名の関数が 4 つ存在)
  * JavaCalls::call_special()            (<= 引数の異なる同名の関数が 4 つ存在)
  * JavaCalls::call_static()             (<= 引数の異なる同名の関数が 4 つ存在)

どのメソッドの場合も, 最終的には StubRoutines::call_stub() が指しているスタブコードを用いて Java メソッドの呼び出しが行われる.
この StubRoutines::call_stub() は,
初期化時に StubGenerator::generate_call_stub() で作られるスタブで,
以下の処理を行うコードが入っている
(1.~3. はダミーの呼び出し元フレーム(Java メソッド風)を構築する処理. スタック上にパラメータを配置し, Gargs などを設定する)
(See: [here](no2935Irx.html) for details).

  1. 新規フレームの作成
  2. フレーム上へのパラメータ(引数)の配置
  3. レジスタ(Gargs 等)の設定
  4. 実際の呼び出し処理

なお, StubRoutines::call_stub() までの呼び出し処理の流れは以下の通り.

  1. どのメソッドの場合も JavaCalls::call() にたどり着く.
  2. JavaCalls::call() から JavaCalls::call_helper() が呼び出される.
  3. JavaCalls::call_helper() の中で, StubRoutines::call_stub() による Java メソッド呼び出しが行われる.


## 備考(Notes)
* StubRoutines::call_stub() のコードを呼び出す際には, 引数として「返り値を格納するアドレス」を渡すことになっている.

  Java メソッドの実行が終わると, 返り値がそのアドレスに格納されて VM ルーチンに渡される.

* StubRoutines::_call_stub_return_address という static 変数に
  上記 "4." の呼び出し命令の直後のアドレス (= Java メソッド呼び出し処理におけるリターンアドレス) が保存されている.

  この変数の値は, 例外処理などでスタックフレームを逆にたどる際に
  call_stub が生成したフレームかどうかの判定(= VMルーチンへの戻り点の検出)に使用されている.

## 備考(Notes)
StubRoutines::call_stub() が作るスタックフレームは以下のような構造になる.

* x86_64 の場合:


```cpp
    ((cite: hotspot/src/cpu/x86/vm/stubGenerator_x86_64.cpp))
      // Call stubs are used to call Java from C
      //
      // Linux Arguments:
      //    c_rarg0:   call wrapper address                   address
      //    c_rarg1:   result                                 address
      //    c_rarg2:   result type                            BasicType
      //    c_rarg3:   method                                 methodOop
      //    c_rarg4:   (interpreter) entry point              address
      //    c_rarg5:   parameters                             intptr_t*
      //    16(rbp): parameter size (in words)              int
      //    24(rbp): thread                                 Thread*
      //
      //     [ return_from_Java     ] <--- rsp
      //     [ argument word n      ]
      //      ...
      // -12 [ argument word 1      ]
      // -11 [ saved r15            ] <--- rsp_after_call
      // -10 [ saved r14            ]
      //  -9 [ saved r13            ]
      //  -8 [ saved r12            ]
      //  -7 [ saved rbx            ]
      //  -6 [ call wrapper         ]
      //  -5 [ result               ]
      //  -4 [ result type          ]
      //  -3 [ method               ]
      //  -2 [ entry point          ]
      //  -1 [ parameters           ]
      //   0 [ saved rbp            ] <--- rbp
      //   1 [ return address       ]
      //   2 [ parameter size       ]
      //   3 [ thread               ]
      //
      // Windows Arguments:
      //    c_rarg0:   call wrapper address                   address
      //    c_rarg1:   result                                 address
      //    c_rarg2:   result type                            BasicType
      //    c_rarg3:   method                                 methodOop
      //    48(rbp): (interpreter) entry point              address
      //    56(rbp): parameters                             intptr_t*
      //    64(rbp): parameter size (in words)              int
      //    72(rbp): thread                                 Thread*
      //
      //     [ return_from_Java     ] <--- rsp
      //     [ argument word n      ]
      //      ...
      // -28 [ argument word 1      ]
      // -27 [ saved xmm15          ] <--- rsp_after_call
      //     [ saved xmm7-xmm14     ]
      //  -9 [ saved xmm6           ] (each xmm register takes 2 slots)
      //  -7 [ saved r15            ]
      //  -6 [ saved r14            ]
      //  -5 [ saved r13            ]
      //  -4 [ saved r12            ]
      //  -3 [ saved rdi            ]
      //  -2 [ saved rsi            ]
      //  -1 [ saved rbx            ]
      //   0 [ saved rbp            ] <--- rbp
      //   1 [ return address       ]
      //   2 [ call wrapper         ]
      //   3 [ result               ]
      //   4 [ result type          ]
      //   5 [ method               ]
      //   6 [ entry point          ]
      //   7 [ parameters           ]
      //   8 [ parameter size       ]
      //   9 [ thread               ]
      //
      //    Windows reserves the callers stack space for arguments 1-4.
      //    We spill c_rarg0-c_rarg3 to this space.
```

* sparc の場合:


```cpp
    ((cite: hotspot/src/cpu/sparc/vm/stubGenerator_sparc.cpp))
        // +---------------+ <--- sp + 0
        // |               |
        // . reg save area .
        // |               |
        // +---------------+ <--- sp + 0x40
        // |               |
        // . extra 7 slots .
        // |               |
        // +---------------+ <--- sp + 0x5c
        // |  empty slot   |      (only if parameter size is even)
        // +---------------+
        // |               |
        // .  parameters   .
        // |               |
        // +---------------+ <--- fp + 0
        // |               |
        // . reg save area .
        // |               |
        // +---------------+ <--- fp + 0x40
        // |               |
        // . extra 7 slots .
        // |               |
        // +---------------+ <--- fp + 0x5c
        // |  param. size  |
        // +---------------+ <--- fp + 0x60
        // |    thread     |
        // +---------------+
        // |               |
```

## 処理の流れ (概要)(Execution Flows : Summary)
### JavaCalls::call() の処理
```
JavaCalls::call()
-> os::os_exception_wrapper()
   -> JavaCalls::call_helper()
      -> (1) 必要があれば JIT コンパイルを開始させる.
             -> CompilationPolicy::must_be_compiled()
             -> CompileBroker::compile_method()
                -> (See: [here](no6HzyuMVW.html) for details)
         
         (1) メソッドのエントリポイントを取得する
             -> methodOopDesc::from_interpreted_entry()
             -> methodOopDesc::interpreter_entry()

         (1) スタック上の Yellow Page が無効にされていたら, 有効化しておく
             -> JavaThread::reguard_stack()

         (1) スタック上の空き領域が shadow page として十分かどうかを確認し, 十分な物理メモリも割り当てておく (空き領域が足りなければ StackOverflowError)
             -> os::stack_shadow_pages_available()
             -> os::bang_stack_shadow_pages()

         (1) 指定された Java メソッドの呼び出しを行う
             -> JavaCallWrapper::JavaCallWrapper()
             -> StubRoutines::call_stub() が指しているコード (= StubGenerator::generate_call_stub() が生成するコード)
                -> (実際の呼び出し処理)
                   -> (実際の呼び出し処理の後, 引き続いて通常のメソッド呼び出し処理が行われる) (See: [here](nop2rLhpmY.html) for details)
             -> JavaCallWrapper::~JavaCallWrapper()
```


### JavaCalls::call_default_constructor() の処理
```
JavaCalls::call_default_constructor()
-> JavaCalls::call()
   -> (同上)
```

### JavaCalls::call_virtual() の処理
```
JavaCalls::call_virtual()
-> LinkResolver::resolve_virtual_call()
   -> (See: [here](no7882NqI.html) for details)
-> JavaCalls::call()
   -> (同上)
```

### JavaCalls::call_special() の処理
```
JavaCalls::call_special()
-> LinkResolver::resolve_special_call()
   -> (See: [here](no7882NqI.html) for details)
-> JavaCalls::call()
   -> (同上)
```

### JavaCalls::call_static() の処理
```
JavaCalls::call_static()
-> LinkResolver::resolve_static_call()
   -> (See: [here](no7882NqI.html) for details)
-> JavaCalls::call()
   -> (同上)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### JavaCalls::call()
See: [here](no3059UMJ.html) for details
### os::os_exception_wrapper() (Linux の場合)
See: [here](no3059hWP.html) for details
### os::os_exception_wrapper() (Solaris の場合)
See: [here](no3059ugV.html) for details
### os::os_exception_wrapper() (Windows の場合)
See: [here](no3059hdD.html) for details
### JavaCalls::call_helper()
See: [here](no30597qb.html) for details
### CompilationPolicy::must_be_compiled()
(#Under Construction)
See: [here](no3059I1h.html) for details
### methodOopDesc::from_interpreted_entry()
See: [here](no3083KSv.html) for details
### methodOopDesc::interpreter_entry()
See: [here](no3083WwK.html) for details
### os::stack_shadow_pages_available()
See: [here](no3059IDK.html) for details
### os::bang_stack_shadow_pages() (Linux の場合)
See: [here](no3059vao.html) for details
### os::bang_stack_shadow_pages() (Solaris の場合)
See: [here](no30598ku.html) for details
### os::bang_stack_shadow_pages() (Windows の場合)
See: [here](no3059Jv0.html) for details
### os::current_stack_pointer() (Windows の場合)
See: [here](no305974D.html) for details
### JavaCallWrapper::JavaCallWrapper()
See: [here](no1904ejQ.html) for details
### JavaCallWrapper::~JavaCallWrapper() 
See: [here](no1904rtW.html) for details
### StubGenerator::generate_call_stub() (sparc の場合)
See: [here](no3059V_n.html) for details
### StubGenerator::generate_call_stub() (x86_64 の場合)
See: [here](no3059VGc.html) for details

### JavaCalls::call_default_constructor()
See: [here](no3059vT0.html) for details

### JavaCalls::call_virtual()
See: [here](no3059Iut.html) for details

### JavaCalls::call_special()
See: [here](no3059V4z.html) for details

### JavaCalls::call_static()
See: [here](no3059HCD.html) for details






