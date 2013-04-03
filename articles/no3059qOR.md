---
layout: default
title: Exception の処理 ： 処理の詳細 (2)&(3) ： 例外オブジェクトの生成処理 & 例外の送出処理 ： ランタイム内での処理  
---
[Up](noHNONT0aT.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (2)&(3) ： 例外オブジェクトの生成処理 & 例外の送出処理 ： ランタイム内での処理  

--- 
## 概要(Summary)
(概要は別項参照 (See: [here](no3059qHd.html) for details))

HotSpot 内には, 例外オブジェクトの生成と送出処理を行うために, 以下のような関数が用意されている.

  * 例外オブジェクト(exception の oop)を生成する関数: 指定された java.lang.Throwable (のサブクラス) のインスタンスを allocate して `<init>` を呼び出す

    Exceptions::new_exception()

  * 例外を送出する関数: (<= 送出というか pending_exception にセットするだけだが...)

    Exceptions::_throw()
    Exceptions::_throw_msg()
    ...

また, 上記の関数の便利なラッパーとして以下のようなマクロも用意されている.

  * 例外送出用のマクロ: (<= 送出というか, Exceptions::_throw で pending_exception をセットしてから return するだけだけど...)


```
    ((cite: hotspot/src/share/vm/utilities/exceptions.hpp))
    // The THROW... macros should be used to throw an exception. They require a THREAD variable to be
    // visible within the scope containing the THROW. Usually this is achieved by declaring the function
    // with a TRAPS argument.
    
    #define THREAD_AND_LOCATION                      THREAD, __FILE__, __LINE__
    
    #define THROW_OOP(e)                                \
      { Exceptions::_throw_oop(THREAD_AND_LOCATION, e);                             return;  }
    
    #define THROW_HANDLE(e)                                \
      { Exceptions::_throw(THREAD_AND_LOCATION, e);                             return;  }
    
    #define THROW(name)                                 \
      { Exceptions::_throw_msg(THREAD_AND_LOCATION, name, NULL); return;  }
    
    #define THROW_MSG(name, message)                    \
      { Exceptions::_throw_msg(THREAD_AND_LOCATION, name, message); return;  }
    
    #define THROW_MSG_LOADER(name, message, loader, protection_domain) \
      { Exceptions::_throw_msg(THREAD_AND_LOCATION, name, message, loader, protection_domain); return;  }
    
    #define THROW_ARG(name, signature, args) \
      { Exceptions::_throw_args(THREAD_AND_LOCATION, name, signature, args);   return; }
    
    #define THROW_OOP_(e, result)                       \
      { Exceptions::_throw_oop(THREAD_AND_LOCATION, e);                           return result; }
    
    #define THROW_HANDLE_(e, result)                       \
      { Exceptions::_throw(THREAD_AND_LOCATION, e);                           return result; }
    
    #define THROW_(name, result)                        \
      { Exceptions::_throw_msg(THREAD_AND_LOCATION, name, NULL); return result; }
    
    #define THROW_MSG_(name, message, result)           \
      { Exceptions::_throw_msg(THREAD_AND_LOCATION, name, message); return result; }
    
    #define THROW_MSG_LOADER_(name, message, loader, protection_domain, result) \
      { Exceptions::_throw_msg(THREAD_AND_LOCATION, name, message, loader, protection_domain); return result; }
    
    #define THROW_ARG_(name, signature, args, result) \
      { Exceptions::_throw_args(THREAD_AND_LOCATION, name, signature, args); return result; }
    
    #define THROW_MSG_CAUSE_(name, message, cause, result)   \
      { Exceptions::_throw_msg_cause(THREAD_AND_LOCATION, name, message, cause); return result; }
    
    
    #define THROW_OOP_0(e)                      THROW_OOP_(e, 0)
    #define THROW_HANDLE_0(e)                   THROW_HANDLE_(e, 0)
    #define THROW_0(name)                       THROW_(name, 0)
    #define THROW_MSG_0(name, message)          THROW_MSG_(name, message, 0)
    #define THROW_WRAPPED_0(name, oop_to_wrap)  THROW_WRAPPED_(name, oop_to_wrap, 0)
    #define THROW_ARG_0(name, signature, arg)   THROW_ARG_(name, signature, arg, 0)
    #define THROW_MSG_CAUSE_0(name, message, cause) THROW_MSG_CAUSE_(name, message, cause, 0)
    
    #define THROW_NULL(name)                    THROW_(name, NULL)
    #define THROW_MSG_NULL(name, message)       THROW_MSG_(name, message, NULL)
```


これらの関数／マクロは以下のように用いられる.

  1. 例外オブジェクトを取得する.

     Exceptions::new_exception() で新しく例外オブジェクトを生成する, 
     もしくは既に作成済みの例外オブジェクトを取得する.

     (IllegalMonitorStateException 等の JVM 標準のものは初期化時に作られている)

  2. 例外送出(pending_exception フィールドへの格納)を行い, 明示的にリターンする (例外を出しただけではリターンはされない)

     Exceptions::_throw_*() を呼んだ後でリターンする.
     格納とリターンをまとめて行う THROW マクロを使ってもいい.


## 備考(Notes)
上記の関数／マクロが使用されるのは, ランタイム中に例外送出条件が満たされた場合 (See: [here](noe6OWoPNB.html) for details) だけではない.
signal handler で検出した場合や明示的なチェックで検出した場合においても, 最終的には上記の関数／マクロを呼び出すことで例外オブジェクトの生成や送出が行われる.


## 処理の流れ (概要)(Execution Flows : Summary)
### Exceptions::new_exception() の処理
```
Exceptions::new_exception()
-> instanceKlass::allocate_instance_handle()
-> JavaCalls::call_special()
   -> (See: [here](no3059iJu.html) for details)
      -> 例外オブジェクトのコンストラクタ(<init> メソッド)
-> JavaCalls::call_special()  (<= cause を設定する必要があれば呼び出す)
   -> (See: [here](no3059iJu.html) for details)
      -> java.lang.Throwable.initCause()
```

### Exceptions::_throw_*() の処理
```
Exceptions::_throw()
-> ThreadShadow::set_pending_exception()
```

```
Exceptions::_throw_msg()
-> Exceptions::new_exception()
   -> (同上)
-> Exceptions::_throw()
   -> (同上)
```

### THROW_* マクロの処理
```
THROW_HANDLE()
-> Exceptions::_throw()
   -> (同上)
```

```
THROW_MSG()
-> Exceptions::_throw_msg()
   -> (同上)
```

```
THROW_OOP_()
-> Exceptions::_throw_oop()
   -> 
```

```
THROW_MSG_LOADER_()
-> Exceptions::_throw_msg()
   -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### Exceptions::new_exception()
See: [here](no30594L2.html) for details
### Exceptions::new_exception()
See: [here](no3059qVF.html) for details
### Exceptions::_throw()
See: [here](no3059R0X.html) for details
### Exceptions::special_exception()
See: [here](no30593fL.html) for details
### Exceptions::_throw_msg()
See: [here](no3059EqR.html) for details
### Exceptions::special_exception()
See: [here](no3059e-d.html) for details
### THROW_MSG()
See: [here](no3059rIk.html) for details
### THROW_HANDLE()
See: [here](no30594Sq.html) for details
### THROW_OOP_()
(#Under Construction)

### THROW_MSG_LOADER_()
(#Under Construction)







