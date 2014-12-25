---
layout: default
title: Serviceability 機能 ： Dynamic Attach (Attach API) 
---
[Up](noOQc_VTg2.html) [Top](../index.html)

#### Serviceability 機能 ： Dynamic Attach (Attach API) 

--- 
## 概要(Summary)
HotSpot 内部では AttachListener クラス (および OS 依存の補助クラス) によって実現されている. 関連するクラス群は以下の通り.

  * AttachListener

    Dynamic Attach に関する機能を納めた名前空間 (AllStatic クラス).
    実際の処理(特にクライアントから要求を受け取る処理)は OS 依存の補助クラスに任せているところも多い.

      * OS 依存の補助クラス (LinuxAttachListener or SolarisAttachListener or Win32AttachListener)
        
        socket を使った client との通信処理部分を AttachListener 本体から隠蔽するためのクラス.

  * AttachOperation

    Dynamic Attach 機能経由での client から HotSpot への要求内容を表すクラス.
    (client から送られてきたリクエスト1個につき1個の AttachOperation オブジェクトが作られる.)

      * OS 依存の補助クラス (LinuxAttachOperation or SolarisAttachOperation or Win32AttachOperation)
        
        OS 依存な情報を追加した AttachOperation.
        (例えば Linux 版では, 通信している client とのコネクション(ソケット)を保持するフィールドが追加されている)

処理としては, リクエスト処理専用のスレッド (以下 AttachListener スレッド) を立ち上げ, 
そのスレッドがクライアントからの要求に応じて処理を行うことで実現されている.
この AttachListener スレッドは, 以下のどちらかの条件が成り立つと生成される
(ただし, "Dynamic Attach" 機能を無効にする DisableAttachMechanism というオプションが存在する.
これをセットした場合は, 以下の条件にかかわらず AttachListener スレッドは生成されなくなる)

* -XX:+StartAttachListener オプションが指定されていた場合, 
  もしくは AttachListener::init_at_startup() による判定が true を返した場合:
  
  この場合, HotSpot の起動時に AttachListener スレッドの生成処理が行われる
  (See: Threads::create_vm()).
  
  Linux や Solaris 環境では
  -XX:+ReduceSignalUsage オプションが指定されていると AttachListener::init_at_startup() が true になる.
  Windows 環境では NT 系列 (Windows NT/2000/XP etc) であれば true になる.

* プラットフォーム依存な条件を満たした状態で HotSpot に SIGBREAK(SIGQUIT) が投げられた場合:
  
  この場合, HotSpot の実行中に AttachListener スレッドの生成処理が行われる
  (See: signal_thread_entry())
  
  Linux や Solaris 環境では, 
  ".attach_pid${pid}" という名前のファイル(ファイルの中身は何でもよい)がカレントディレクトリか /tmp にある状態で SIGQUIT を投げれば生成される (See: AttachListener::is_init_trigger())).
  
  Windows 環境では, 起動後に AttachListener スレッドを生成する手段はない
  (See: AttachListener::is_init_trigger())).


AttachListener スレッドの実際の処理は attach_listener_thread_entry() という関数に実装されている.
AttachListener スレッドはクライアントプロセスからの接続用のソケットを開いて待機し, 
クライアントからの要求が来るとそれに応じた処理を行って結果を返す.

要求を出すクライアント側は
com.sun.tools.attach.VirtualMachine という Java クラスを使用する.
この com.sun.tools.attach.VirtualMachine クラスは実際には abstract クラスであり, 
sun.tools.attach.HotSpotVirtualMachine というクラスに継承され (ただしこいつも abstract クラス),
最終的には各 OS ごとのクラス (LinuxVirtualMachine, SolarisVirtualMachine, WindowsVirtualMachine) で実際の処理が実装されている.


## 備考(Notes)
AttachListener スレッドが認識可能なコマンド, 及びそのコマンドに対応して呼ばれる関数は以下の通り.

### 全環境共通のコマンド
<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table17766Okw -->
| Command | Function | Note |
|---|---|---|
| agentProperties | get_agent_properties() |  |
| datadump | data_dump() |  |
| dumpheap | dump_heap() |  |
| load | JvmtiExport::load_agent_library() | 新しい JVMTI エージェントをロードする処理 |
| properties | get_system_properties() |  |
| threaddump | thread_dump() |  |
| inspectheap | heap_inspection() |  |
| setflag | set_flag() |  |
| printflag | print_flag() |  |
<!-- END RECEIVE ORGTBL table17766Okw -->

<!-- 
#+ORGTBL: SEND table17766Okw orgtbl-to-gfm :no-escape t
| Command         | Function                          | Note                                      |
|-----------------+-----------------------------------+-------------------------------------------|
| agentProperties | get_agent_properties()            |                                           |
| datadump        | data_dump()                       |                                           |
| dumpheap        | dump_heap()                       |                                           |
| load            | JvmtiExport::load_agent_library() | 新しい JVMTI エージェントをロードする処理 |
| properties      | get_system_properties()           |                                           |
| threaddump      | thread_dump()                     |                                           |
| inspectheap     | heap_inspection()                 |                                           |
| setflag         | set_flag()                        |                                           |
| printflag       | print_flag()                      |                                           |
-->


```cpp
    ((cite: hotspot/src/share/vm/services/attachListener.cpp))
    // Table to map operation names to functions.
    
    // names must be of length <= AttachOperation::name_length_max
    static AttachOperationFunctionInfo funcs[] = {
      { "agentProperties",  get_agent_properties },
      { "datadump",         data_dump },
    #ifndef SERVICES_KERNEL
      { "dumpheap",         dump_heap },
    #endif  // SERVICES_KERNEL
      { "load",             JvmtiExport::load_agent_library },
      { "properties",       get_system_properties },
      { "threaddump",       thread_dump },
      { "inspectheap",      heap_inspection },
      { "setflag",          set_flag },
      { "printflag",        print_flag },
      { NULL,               NULL }
    };
```

### Linux 環境限定のコマンド 
(無し)

```cpp
    ((cite: hotspot/src/os/linux/vm/attachListener_linux.cpp))
    AttachOperationFunctionInfo* AttachListener::pd_find_operation(const char* n) {
      return NULL;
    }
```

### Solaris 環境限定のコマンド
<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table17766bu2 -->
| Command | Function | Note |
|---|---|---|
| enabledprobes | enable_dprobes() |  |
<!-- END RECEIVE ORGTBL table17766bu2 -->

<!-- 
#+ORGTBL: SEND table17766bu2 orgtbl-to-gfm :no-escape t
| Command       | Function         | Note |
|---------------+------------------+------|
| enabledprobes | enable_dprobes() |      |
-->


```cpp
    ((cite: hotspot/src/os/solaris/vm/attachListener_solaris.cpp))
    // platform specific operations table
    static AttachOperationFunctionInfo funcs[] = {
      { "enabledprobes", enable_dprobes },
      { NULL, NULL }
    };
```

### Windows 環境限定のコマンド 
(無し)

```cpp
    ((cite: hotspot/src/os/windows/vm/attachListener_windows.cpp))
    AttachOperationFunctionInfo* AttachListener::pd_find_operation(const char* n) {
      return NULL;
    }
```


## 備考(Notes)
上記の AttachListener スレッドが認識可能なコマンドのうち, 
com.sun.tools.attach.VirtualMachine クラスから呼び出せるのは 
"load", "properties", "agentProperties" のみ
(See: com.sun.tools.attach.VirtualMachine.loadAgent(), com.sun.tools.attach.VirtualMachine.getSystemProperties(), com.sun.tools.attach.VirtualMachine.getAgentProperties()).

その他のコマンドは (非公式なクラスである)
sun.tools.attach.HotSpotVirtualMachine クラスからのみ呼び出せる.

## 備考(Notes)
com.sun.tools.attach.VirtualMachine クラスは, 
内部で com.sun.tools.attach.spi.AttachProvider という補助クラスを使用する.

このクラスも実際には abstract クラスであり, 
sun.tools.attach.HotSpotAttachProvider というクラスに継承され (ただしこいつも abstract クラス),
最終的には各 OS ごとのクラス (LinuxAttachProvider, SolarisAttachProvider, WindowsAttachProvider) で実際の処理が実装されている.

## 処理の流れ (概要)(Execution Flows : Summary)
### AttachListener スレッドの生成処理 (HotSpot の起動時の場合)
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> AttachListener::init_at_startup()
   -> AttachListener::init()
      -> JavaThread::JavaThread()   (<= なお, エントリポイントとしては attach_listener_thread_entry() 関数が指定されている)
         -> (See: [here](no2935KMw.html) for details)
      -> Thread::start()
         -> (See: [here](no2935KMw.html) for details)
```

### AttachListener スレッドの生成処理 (HotSpot の実行中の場合) (&& Linux 又は Solaris の場合)
```
(See: [here](no28916GhX.html) for details)
-> signal_thread_entry()
   -> AttachListener::is_init_trigger()
      -> AttachListener::init()
         -> (同上)
```

### 生成された AttachListener スレッドの処理
```
attach_listener_thread_entry()
-> (1) 通信処理用の初期化を行う
       -> AttachListener::pd_init()  (<= このメソッドは各サブクラスでオーバーライドされている)
          -> * Linux の場合:
               -> LinuxAttachListener::init()
                  -> (通信用の unix domain socket を作成)
             * Solaris の場合:
               -> SolarisAttachListener::init()
                  -> SolarisAttachListener::create_door()
                     -> (通信用の door を作成) (door のエントリポイントは enqueue_proc())
             * Windows の場合:
               -> WindowsAttachListener::init()

   (1) 以下の処理を無限ループで繰り返す.
       (1) クライアントからの新しい要求を取得する
           -> AttachListener::dequeue()  (<= このメソッドは各サブクラスでオーバーライドされている)
              -> * Linux の場合:
                   -> LinuxAttachListener::dequeue()
                      -> LinuxAttachListener::read_request()
                 * Solaris の場合:
                   -> SolarisAttachListener::dequeue()
                      -> SolarisAttachListener::head()
                 * Windows の場合:
                   -> Win32AttachListener::dequeue()

       (1) 要求に対応する関数を呼び出す.
           -> AttachListener::detachall()
              -> * Linux の場合:
                   -> AttachListener::pd_detachall()
                 * Solaris の場合:
                   -> AttachListener::pd_detachall()
                      -> DTrace::detach_all_clients()
                 * Windows の場合:
                   -> AttachListener::pd_detachall()
           -> get_agent_properties()
           -> data_dump()
           -> dump_heap()
           -> JvmtiExport::load_agent_library()
              -> os::dll_build_name()
              -> os::dll_load()
              -> os::dll_lookup()
              -> (ロードしたライブラリの Agent_OnAttach() を呼び出す)
           -> get_system_properties()
           -> thread_dump()
           -> heap_inspection()
           -> set_flag()
           -> print_flag()

       (1) クライアントに結果を送信する
           -> AttachOperation::complete()  (<= このメソッドは各サブクラスでオーバーライドされている)
              -> * Linux の場合:
                   -> LinuxAttachOperation::complete()
                 * Solaris の場合:
                   -> SolarisAttachOperation::complete()
                 * Windows の場合:
                   -> Win32AttachOperation::complete()
```

#### door が呼び出された際の処理 (Solaris の場合)
```
enqueue_proc()
-> create_operation()
-> SolarisAttachListener::enqueue()
```


### クライアントプロセスによる呼び出し処理
#### com.sun.tools.attach.VirtualMachine.attach(String id) の処理
```
com.sun.tools.attach.VirtualMachine.attach(String id)
-> com.sun.tools.attach.spi.AttachProvider.providers()
   -> 
-> com.sun.tools.attach.spi.AttachProvider.attachVirtualMachine()
   -> 
```

#### com.sun.tools.attach.VirtualMachine.detach() の処理
(このメソッドは各サブクラスでオーバーライドされている)

```
* Linux の場合:
  sun.tools.attach.LinuxVirtualMachine.detach()

* Solaris の場合:
  sun.tools.attach.SolarisVirtualMachine.detach()

* Windows の場合:
  sun.tools.attach.WindowsVirtualMachine.detach()
```

#### com.sun.tools.attach.VirtualMachine.loadAgent() の処理
```
sun.tools.attach.HotSpotVirtualMachine.loadAgent()
-> sun.tools.attach.HotSpotVirtualMachine.loadAgentLibrary(String agentLibrary, String options)  (<= 引数は "instrument" (libinstrumentの意))
   -> sun.tools.attach.HotSpotVirtualMachine.loadAgentLibrary(String agentLibrary, boolean isAbsolute, String options)
      -> (1) コマンドとして "load" を指定して execute() を呼び出す
             -> sun.tools.attach.HotSpotVirtualMachine.execute() (<= このメソッドは各サブクラスでオーバーライドされている)
                -> * Linux の場合:
                     -> sun.tools.attach.LinuxVirtualMachine.execute()
                        -> (unix domain socket 経由で実行させたいコマンド(およびその引数)を通知)
                   * Solaris の場合:
                     -> sun.tools.attach.SolarisVirtualMachine.execute()
                        -> (door 経由で実行させたいコマンド(およびその引数)を通知)
                   * Windows の場合:
                     -> sun.tools.attach.WindowsVirtualMachine.execute()
                        -> 
```

#### com.sun.tools.attach.VirtualMachine.getSystemProperties() の処理
```
sun.tools.attach.HotSpotVirtualMachine.getSystemProperties()
-> sun.tools.attach.HotSpotVirtualMachine.executeCommand() (<= 引数は "properties")
   -> sun.tools.attach.HotSpotVirtualMachine.execute()
      -> (同上)
```

#### com.sun.tools.attach.VirtualMachine.getAgentProperties() の処理
```
sun.tools.attach.HotSpotVirtualMachine.getAgentProperties()
-> sun.tools.attach.HotSpotVirtualMachine.executeCommand() (<= 引数は "agentProperties")
   -> sun.tools.attach.HotSpotVirtualMachine.execute()
      -> (同上)
```

#### sun.tools.attach.HotSpotVirtualMachine.localDataDump() の処理
```
sun.tools.attach.HotSpotVirtualMachine.localDataDump()
-> sun.tools.attach.HotSpotVirtualMachine.executeCommand() (<= 引数は "datadump")
   -> sun.tools.attach.HotSpotVirtualMachine.execute()
      -> (同上)
```

#### sun.tools.attach.HotSpotVirtualMachine.remoteDataDump() の処理
```
sun.tools.attach.HotSpotVirtualMachine.remoteDataDump()
-> sun.tools.attach.HotSpotVirtualMachine.executeCommand() (<= 引数は "threaddump")
   -> sun.tools.attach.HotSpotVirtualMachine.execute()
      -> (同上)
```

#### sun.tools.attach.HotSpotVirtualMachine.dumpHeap() の処理
```
sun.tools.attach.HotSpotVirtualMachine.dumpHeap()
-> sun.tools.attach.HotSpotVirtualMachine.executeCommand() (<= 引数は "dumpheap")
   -> sun.tools.attach.HotSpotVirtualMachine.execute()
      -> (同上)
```

#### sun.tools.attach.HotSpotVirtualMachine.heapHisto() の処理
```
sun.tools.attach.HotSpotVirtualMachine.heapHisto()
-> sun.tools.attach.HotSpotVirtualMachine.executeCommand() (<= 引数は "inspectheap")
   -> sun.tools.attach.HotSpotVirtualMachine.execute()
      -> (同上)
```

#### sun.tools.attach.HotSpotVirtualMachine.setFlag() の処理
```
sun.tools.attach.HotSpotVirtualMachine.setFlag()
-> sun.tools.attach.HotSpotVirtualMachine.executeCommand() (<= 引数は "setflag")
   -> sun.tools.attach.HotSpotVirtualMachine.execute()
      -> (同上)
```

#### sun.tools.attach.HotSpotVirtualMachine.printFlag() の処理
```
sun.tools.attach.HotSpotVirtualMachine.printFlag()
-> sun.tools.attach.HotSpotVirtualMachine.executeCommand() (<= 引数は "printflag")
   -> sun.tools.attach.HotSpotVirtualMachine.execute()
      -> (同上)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### AttachListener::init_at_startup()  (Linux の場合)
See: [here](no17766gU0.html) for details
### AttachListener::init_at_startup()  (Solaris の場合)
See: [here](no17766GHc.html) for details
### AttachListener::init_at_startup()  (Windows の場合)
See: [here](no17766gic.html) for details

### AttachListener::init()
See: [here](no17766tGJ.html) for details

### AttachListener::is_init_trigger()  (Linux の場合)
See: [here](no17766um1.html) for details
### AttachListener::is_init_trigger()  (Solaris の場合)
See: [here](no1776673v.html) for details
### AttachListener::is_init_trigger()  (Windows の場合)
See: [here](no17766HWL.html) for details

### attach_listener_thread_entry()
See: [here](no17766esM.html) for details

### AttachListener::pd_init()  (Linux の場合)
See: [here](no17766RG2.html) for details
### LinuxAttachListener::init()  (Linux の場合)
See: [here](no17766rh2.html) for details
### AttachListener::pd_init()  (Solaris の場合)
See: [here](no177669hM.html) for details
### SolarisAttachListener::init()  (Solaris の場合)
See: [here](no17766kAf.html) for details
### SolarisAttachListener::create_door()  (Solaris の場合)
See: [here](no17766Lfx.html) for details
### AttachListener::pd_init()  (Windows の場合)
See: [here](no17766vuw.html) for details
### WindowsAttachListener::init()  (Windows の場合)
(#Under Construction)
See: [here](no177667MM.html) for details

### enqueue_proc()  (Solaris の場合)
(#Under Construction)
See: [here](no17766HDZ.html) for details
### create_operation()  (Solaris の場合)
(#Under Construction)

### SolarisAttachListener::enqueue()  (Solaris の場合)
(#Under Construction)


### AttachListener::dequeue()  (Linux の場合)
See: [here](no17766JmB.html) for details
### LinuxAttachListener::dequeue()  (Linux の場合)
(#Under Construction)
See: [here](no177669VO.html) for details
### LinuxAttachListener::read_request()  (Linux の場合)
(#Under Construction)
See: [here](no177669JQ.html) for details

### AttachListener::dequeue()  (Solaris の場合)
See: [here](no17766wEU.html) for details
### SolarisAttachListener::dequeue()  (Solaris の場合)
See: [here](no17766H8k.html) for details
### AttachListener::dequeue()  (Windows の場合)
See: [here](no17766Xjm.html) for details
### Win32AttachListener::dequeue()  (Windows の場合)
(#Under Construction)


### AttachListener::detachall()
See: [here](no17766u5n.html) for details
### AttachListener::pd_detachall()  (Linux の場合)
See: [here](no177666j2.html) for details
### AttachListener::pd_detachall()  (Solaris の場合)
See: [here](no17766GCS.html) for details
### DTrace::detach_all_clients()  (Solaris の場合)
See: [here](no31150lih.html) for details
### DTrace::disable_dprobes()  (Solaris の場合)
See: [here](no31150YYb.html) for details
### DTrace::enable_dprobes()  (Solaris の場合)
See: [here](no2114N1E.html) for details
### AttachListener::pd_detachall()  (Windows の場合)
See: [here](no17766H1w.html) for details

### JvmtiExport::load_agent_library()
See: [here](no17766_Eq.html) for details
### os::dll_build_name()  (Linux の場合)
(#Under Construction)

### os::dll_build_name()  (Solaris の場合)
(#Under Construction)

### os::dll_build_name()  (Windows の場合)
(#Under Construction)

### os::dll_load()  (Linux の場合)
See: [here](no17119KQT.html) for details
### os::dll_load()  (Solaris の場合)
See: [here](no17119XaZ.html) for details
### os::dll_load()  (Windows の場合)
See: [here](no3059djw.html) for details
### os::dll_lookup()  (Linux の場合)
See: [here](no17119WZS.html) for details
### os::dll_lookup()  (Solaris の場合)
See: [here](no17119jjY.html) for details
### os::dll_lookup()  (Windows の場合)
See: [here](no3059qt2.html) for details

### LinuxAttachOperation::complete()
(#Under Construction)

### SolarisAttachOperation::complete()
(#Under Construction)

### Win32AttachOperation::complete()
(#Under Construction)


### listener_cleanup()  (Linux の場合)
(#Under Construction)

### listener_cleanup()  (Solaris の場合)
(#Under Construction)


### com.sun.tools.attach.VirtualMachine.attach(String id)
(#Under Construction)
See: [here](no17766Cbx.html) for details

### sun.tools.attach.HotSpotVirtualMachine.loadAgent()
See: [here](no17766xFb.html) for details
### sun.tools.attach.HotSpotVirtualMachine.loadAgentLibrary(String agentLibrary, String options)
See: [here](no17766xMP.html) for details
### sun.tools.attach.HotSpotVirtualMachine.loadAgentLibrary(String agentLibrary, boolean isAbsolute, String options)
See: [here](no17766Yrh.html) for details
### sun.tools.attach.LinuxVirtualMachine.execute()
See: [here](no17766_Xc.html) for details
### sun.tools.attach.SolarisVirtualMachine.execute()
See: [here](no177662Ym.html) for details
### sun.tools.attach.WindowsVirtualMachine.execute()
(#Under Construction)
See: [here](no17766E5P.html) for details

### sun.tools.attach.LinuxVirtualMachine.detach()
See: [here](no17766sR1.html) for details
### sun.tools.attach.SolarisVirtualMachine.detach()
See: [here](no17766Gt1.html) for details
### sun.tools.attach.WindowsVirtualMachine.detach()
(#Under Construction)
See: [here](no17766fcL.html) for details

### sun.tools.attach.HotSpotVirtualMachine.executeCommand()
See: [here](no17766zHp.html) for details
### sun.tools.attach.HotSpotVirtualMachine.getSystemProperties()
See: [here](no17766lKE.html) for details
### sun.tools.attach.HotSpotVirtualMachine.getAgentProperties()
See: [here](no17766MpW.html) for details
### sun.tools.attach.HotSpotVirtualMachine.localDataDump()
See: [here](no17766a0j.html) for details
### sun.tools.attach.HotSpotVirtualMachine.remoteDataDump()
See: [here](no17766n-p.html) for details
### sun.tools.attach.HotSpotVirtualMachine.dumpHeap()
See: [here](no17766BT2.html) for details
### sun.tools.attach.HotSpotVirtualMachine.heapHisto()
See: [here](no17766zcF.html) for details
### sun.tools.attach.HotSpotVirtualMachine.setFlag()
See: [here](no17766AnL.html) for details
### sun.tools.attach.HotSpotVirtualMachine.printFlag()
See: [here](no17766NxR.html) for details






