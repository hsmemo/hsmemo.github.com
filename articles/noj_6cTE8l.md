---
layout: default
title: JniPeriodicChecker クラス関連のクラス (JniPeriodicChecker, 及びその補助クラス(JniPeriodicCheckerTask))
---
[Top](../index.html)

#### JniPeriodicChecker クラス関連のクラス (JniPeriodicChecker, 及びその補助クラス(JniPeriodicCheckerTask))

これらは, トラブルシューティング用のクラス.
より具体的に言うと, JNI のネイティブコードが起こしうる問題を定期的に検査するためのクラス.


### クラス一覧(class list)

  * [JniPeriodicChecker](#no21zQkAtP)
  * [JniPeriodicCheckerTask](#noCzckyZmm)


---
## <a name="no21zQkAtP" id="no21zQkAtP">JniPeriodicChecker</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する product オプションが指定されている場合にのみ使用される) 
(See: CheckJNICalls, -Xcheck:jni).

(なお, CheckJNICalls オプションと -Xcheck:jni オプションは同じもの)

JNI のネイティブコードが起こしうる問題を定期的に検査するためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).
例えば, signal handler が上書きされる, zero page のマッピングが上書きされる, 等の問題を検出する.


```
    ((cite: hotspot/src/share/vm/runtime/jniPeriodicChecker.hpp))
    /*
     * This gets activated under Xcheck:jni (CheckJNICalls), and is typically
     * to detect any errors caused by JNI applications, such as signal handler,
     * hijacking, va 0x0 hijacking either by mmap or an OS error.
     */
```


```
    ((cite: hotspot/src/share/vm/runtime/jniPeriodicChecker.hpp))
    class JniPeriodicChecker : AllStatic {
```

### 使われ方(Usage)
Threads::create_vm() 内で(のみ)使用されている.

(CheckJNICalls オプションが指定されている場合, 
 Threads::create_vm() 内で JniPeriodicChecker::engage() が呼ばれて検査処理が開始される)

### 内部構造(Internal structure)
実際の検査処理は JniPeriodicCheckerTask に丸投げしている
(JniPeriodicChecker は JniPeriodicCheckerTask を開始させるクラス).

### 備考(Notes)
JniPeriodicChecker::disengage() というメソッドも提供しているが, これは使用されていない.

(これは JniPeriodicCheckerTask を停止するためのメソッド.
 コメントを読むと before_exit() から呼び出されるように見えるが呼び出されていない... (バグ?? #TODO))


```
    ((cite: hotspot/src/share/vm/runtime/jniPeriodicChecker.cpp))
    /*
     * the disengage() method is responsible for deactivating the periodic
     * task. This  method is called from before_exit() in java.cpp and is only called
     * after the WatcherThread has been stopped.
     */
    void JniPeriodicChecker::disengage() {
```




### 詳細(Details)
See: [here](../doxygen/classJniPeriodicChecker.html) for details

---
## <a name="noCzckyZmm" id="noCzckyZmm">JniPeriodicCheckerTask</a>

### 概要(Summary)
JniPeriodicChecker クラス用の補助クラス.

定期間隔で JNI のネイティブコードが起こしうる問題を検査するためのクラス(PeriodicTaskクラス).


```
    ((cite: hotspot/src/share/vm/runtime/jniPeriodicChecker.cpp))
    // --------------------------------------------------------
    // Class to aid in periodic checking under CheckJNICalls
    class JniPeriodicCheckerTask : public PeriodicTask {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
JniPeriodicChecker クラスの _task フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
JniPeriodicChecker::engage() 内で(のみ)生成されている.

### 内部構造(Internal structure)
定期間隔で os::run_periodic_checks() を呼び出しているだけ.


```
    ((cite: hotspot/src/share/vm/runtime/jniPeriodicChecker.cpp))
         void task() { os::run_periodic_checks(); }
```




### 詳細(Details)
See: [here](../doxygen/classJniPeriodicCheckerTask.html) for details

---
