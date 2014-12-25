---
layout: default
title: os クラスのプラットフォーム依存な補助クラス (os::Linux, os::Linux::SuspendResume, os::PlatformEvent, os::PlatformParker)
---
[Top](../index.html)

#### os クラスのプラットフォーム依存な補助クラス (os::Linux, os::Linux::SuspendResume, os::PlatformEvent, os::PlatformParker)



### クラス一覧(class list)

  * [os::Linux](#nomoWafDFk)
  * [os::Linux::SuspendResume](#nomYIgpLNY)
  * [os::PlatformEvent](#nob8zL_1wC)
  * [os::PlatformParker](#noscD0HayL)


---
## <a name="nomoWafDFk" id="nomoWafDFk">os::Linux</a>

### 概要(Summary)
os クラス内で使用される補助クラス (See: os).

(このクラスが Linux に依存した機能 (システムコール等) を os クラス本体から隠蔽している)


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.hpp))
    class Linux {
```




### 詳細(Details)
See: [here](../doxygen/classos_1_1Linux.html) for details

---
## <a name="nomYIgpLNY" id="nomYIgpLNY">os::Linux::SuspendResume</a>

### 概要(Summary)
Linux 環境における OSThread クラス用の補助クラス (See: OSThread).

signal を用いたスレッドの suspend/resume 処理で使用されるクラス.
現在のスレッドのサスペンド状態(サスペンドされていない, サスペンド状態への移行中, サスペンド中)を表す.


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.hpp))
      // Linux suspend/resume support - this helper is a shadow of its former
      // self now that low-level suspension is barely used, and old workarounds
      // for LinuxThreads are no longer needed.
      class SuspendResume {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Linux 用の OSThread オブジェクトの sr フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/os/linux/vm/osThread_linux.hpp))
      // flags that support signal based suspend/resume on Linux are in a
      // separate class to avoid confusion with many flags in OSThread that
      // are used by VM level suspend/resume.
      os::Linux::SuspendResume sr;
```

#### 使用箇所(where its instances are used)
現在は FlatProfiler によるプロファイル取得処理で(のみ)使用されている.
より具体的に言うと, 以下の箇所で(のみ)使用されている (See: [here](no5248oCL.html) for details).

* resume_clear_context()
* SR_handler()
* do_suspend()
* do_resume()




### 詳細(Details)
See: [here](../doxygen/classos_1_1Linux_1_1SuspendResume.html) for details

---
## <a name="nob8zL_1wC" id="nob8zL_1wC">os::PlatformEvent</a>

### 概要(Summary)
ParkEvent クラスのスーパークラス.
このクラスが実際のスレッド制御機能を提供している (ParkEvent はこのクラスのラッパー的な存在)
(See: ParkEvent) (See: [here](no2114COc.html) for details).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.hpp))
    class PlatformEvent : public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 適当な os::PlatformEvent オブジェクトを用意して os::PlatformEvent::park() を呼ぶと, 
   park() を呼んだスレッドがブロックされる.
2. 別のスレッドがその os::PlatformEvent オブジェクトに対して os::PlatformEvent::unpark() を呼ぶと, ブロックしていたスレッドが起床される.
3. なお, ブロックにタイムアウトを付けたい場合は, 
   os::PlatformEvent::park() の代わりに os::PlatformEvent::park(jlong millis) を使えばいい.

### 内部構造(Internal structure)
内部的には, _Event というフィールドが mutex 代わりになっている.


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.hpp))
        volatile int _Event ;
```

_Event フィールドは 0 か 1 の値を取る.

  * 0 は通常の状態. この状態で park() するとブロックされる.

  * 1 は park() よりも先に unpark() 処理が行われた状態.
    この状態で park() すると (先の unpark() と相殺される形になり) ブロックはされない (単に _Event が 0 になるだけ).

そして, park()/unpark() 処理は以下のように実装されている.

  * park() 操作では, _Event の値を 1つデクリメントする.

    値が 0 未満になってしまったら, 誰かが unpark() してくれるまで待機する
    (待機する方法はプラットフォーム依存で, pthread_cond_wait() だったり WaitForSingleObject() だったりする).

  * unpark() 操作では, _Event の値を 1つインクリメントする (ただし, 現在の値が 1であれば何もしない).

    元の値が負値だった場合は, 待機しているスレッドを起こす
    (起こす方法はプラットフォーム依存で, pthread_cond_signal() だったり SetEvent() だったりする).

また, 現在待機中のスレッド数は _nParked というフィールドに記録されている
(このフィールドも 0 か 1 しか取らない.
 このフィールドが 0 の場合には, unpark() 内での起床処理が省略される).


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.hpp))
        volatile int _nParked ;
```




### 詳細(Details)
See: [here](../doxygen/classos_1_1PlatformEvent.html) for details

---
## <a name="noscD0HayL" id="noscD0HayL">os::PlatformParker</a>

### 概要(Summary)
Parker クラスのスーパークラス.
このクラスが実際のスレッド制御機能を提供している (Parker はこのクラスのラッパー的な存在)
(See: Parker) (See: [here](no2114PYi.html) for details).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.hpp))
    class PlatformParker : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classos_1_1PlatformParker.html) for details

---
