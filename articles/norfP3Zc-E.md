---
layout: default
title: AttachListener クラスのプラットフォーム依存な補助クラス (LinuxAttachListener, LinuxAttachOperation, ArgumentIterator)
---
[Top](../index.html)

#### AttachListener クラスのプラットフォーム依存な補助クラス (LinuxAttachListener, LinuxAttachOperation, ArgumentIterator)

これらは, 保守運用機能のためのクラス.
より具体的に言うと, "Dynamic Attach" 機能のためのクラス (See: [here](no3026gMG.html) for details).

### 備考(Notes)
Linux では AttachListener クラスは UNIX domain socket を使って実装されている.


```cpp
    ((cite: hotspot/src/os/linux/vm/attachListener_linux.cpp))
    // The attach mechanism on Linux uses a UNIX domain socket. An attach listener
    // thread is created at startup or is created on-demand via a signal from
    // the client tool. The attach listener creates a socket and binds it to a file
    // in the filesystem. The attach listener then acts as a simple (single-
    // threaded) server - it waits for a client to connect, reads the request,
    // executes it, and returns the response to the client via the socket
    // connection.
    //
    // As the socket is a UNIX domain socket it means that only clients on the
    // local machine can connect. In addition there are two other aspects to
    // the security:
    // 1. The well known file that the socket is bound to has permission 400
    // 2. When a client connect, the SO_PEERCRED socket option is used to
    //    obtain the credentials of client. We check that the effective uid
    //    of the client matches this process.
```


### クラス一覧(class list)

  * [LinuxAttachListener](#no_YCdS-aL)
  * [LinuxAttachOperation](#nooYKR6MMc)
  * [ArgumentIterator](#no2opgKKtL)


---
## <a name="no_YCdS-aL" id="no_YCdS-aL">LinuxAttachListener</a>

### 概要(Summary)
AttachListener クラス用の関数のうちで Linux に依存したものを納めた名前空間(AllStatic クラス) (See: [here](no3026gMG.html) for details).

(このクラスが Unix domain socket を使った通信処理を AttachListener クラス本体から隠蔽している)


```cpp
    ((cite: hotspot/src/os/linux/vm/attachListener_linux.cpp))
    class LinuxAttachListener: AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classLinuxAttachListener.html) for details

---
## <a name="nooYKR6MMc" id="nooYKR6MMc">LinuxAttachOperation</a>

### 概要(Summary)
AttachOperation クラスの具象サブクラスの1つ (Linux 用の AttachOperation クラス).


```cpp
    ((cite: hotspot/src/os/linux/vm/attachListener_linux.cpp))
    class LinuxAttachOperation: public AttachOperation {
```

### 内部構造(Internal structure)
AttachOperation クラスのフィールドに加えて, client と通信するソケット用のフィールドを保持している
(このクラスがソケットをカプセル化している).


```cpp
    ((cite: hotspot/src/os/linux/vm/attachListener_linux.cpp))
      // the connection to the client
      int _socket;
```




### 詳細(Details)
See: [here](../doxygen/classLinuxAttachOperation.html) for details

---
## <a name="no2opgKKtL" id="no2opgKKtL">ArgumentIterator</a>

### 概要(Summary)
LinuxAttachListener クラス内で使用される補助クラス(StackObjクラス).

client から受信したデータ中の引数を切り出す(パースする)処理を行う.


```cpp
    ((cite: hotspot/src/os/linux/vm/attachListener_linux.cpp))
    // Supporting class to help split a buffer into individual components
    class ArgumentIterator : public StackObj {
```

### 使われ方(Usage)
LinuxAttachListener::read_request() 内で(のみ)使用されている
(なお, LinuxAttachListener::read_request() は
client から送られてきたデータを元に LinuxAttachOperation オブジェクトを生成する関数).

ArgumentIterator によるパース結果は ArgumentIterator::next() で取り出すことができる.


```cpp
    ((cite: hotspot/src/os/linux/vm/attachListener_linux.cpp))
    LinuxAttachOperation* LinuxAttachListener::read_request(int s) {
    ...
      // parse request
    
      ArgumentIterator args(buf, (max_len)-left);
    
      // version already checked
      char* v = args.next();
    
      char* name = args.next();
```




### 詳細(Details)
See: [here](../doxygen/classArgumentIterator.html) for details

---
