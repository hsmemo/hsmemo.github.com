---
layout: default
title: VMError クラス (VMError, 及びその補助クラス(VM_ReportJavaOutOfMemory))
---
[Top](../index.html)

#### VMError クラス (VMError, 及びその補助クラス(VM_ReportJavaOutOfMemory))



### クラス一覧(class list)

  * [VMError](#nofaLdv53m)
  * [VM_ReportJavaOutOfMemory](#noLrIUQ-MN)


---
## <a name="nofaLdv53m" id="nofaLdv53m">VMError</a>

### 概要(Summary)
HotSpot が異常終了する際のエラー通知作業で使われる補助クラス(StackObjクラス).

デフォルトでは, 標準出力 (defaultStream::output_fd()) に簡単なエラーメッセージを出した後, 
エラーレポートファイル ("hs_err_pid<pid>.log") に詳細なエラー内容を出力し, 
その後 HotSpot をアボートさせる.

なお複数のスレッドから同時に使用された場合でも実際の出力処理は一人しか実行できない
(残りのスレッドは VMError::report_and_die() 内でブロックされたままアボートすることになる).


```cpp
    ((cite: hotspot/src/share/vm/utilities/vmError.cpp))
    // Fatal error handler for internal errors and crashes.
    //
    // The default behavior of fatal error handler is to print a brief message
    // to standard out (defaultStream::output_fd()), then save detailed information
    // into an error report file (hs_err_pid<pid>.log) and abort VM. If multiple
    // threads are having troubles at the same time, only one error is reported.
    // The thread that is reporting error will abort VM when it is done, all other
    // threads are blocked forever inside report_and_die().
```


```cpp
    ((cite: hotspot/src/share/vm/utilities/vmError.hpp))
    class VMError : public StackObj {
```

### 使われ方(Usage)
HotSpot 内の様々なエラー発生箇所で VMError::report_and_die() が呼び出されている
(この関数内で端末や hs_err_pid*.log ファイルへの出力が行われる).


```cpp
    ((cite: hotspot/src/share/vm/utilities/vmError.hpp))
      // main error reporting function
      void report_and_die();
```

### 備考(Notes)
出力先は OnError オプションで変更することもできる.

(他のコマンドにパイプで出力をつなぐ等, 少し凝ったこともできる模様)


```cpp
    ((cite: hotspot/src/share/vm/utilities/vmError.cpp))
    // -XX:OnError=<string>, where <string> can be a list of commands, separated
    // by ';'. "%p" is replaced by current process id (pid); "%%" is replaced by
    // a single "%". Some examples:
    //
    // -XX:OnError="pmap %p"                // show memory map
    // -XX:OnError="gcore %p; dbx - %p"     // dump core and launch debugger
    // -XX:OnError="cat hs_err_pid%p.log | mail my_email@sun.com"
    // -XX:OnError="kill -9 %p"             // ?#!@#
```




### 詳細(Details)
See: [here](../doxygen/classVMError.html) for details

---
## <a name="noLrIUQ-MN" id="noLrIUQ-MN">VM_ReportJavaOutOfMemory</a>

### 概要(Summary)
VMError クラス内で使用される補助クラス.

OnOutOfMemoryError オプションで指定されたコマンドを fork-and-exec するためのクラス
(なお, OnOutOfMemoryError オプションは OutOfMemoryError が起きた際に実行するコマンドを指定するオプション).


```cpp
    ((cite: hotspot/src/share/vm/utilities/vmError.cpp))
    /*
     * OnOutOfMemoryError scripts/commands executed while VM is a safepoint - this
     * ensures utilities such as jmap can observe the process is a consistent state.
     */
    class VM_ReportJavaOutOfMemory : public VM_Operation {
```

### 使われ方(Usage)
VMError::report_java_out_of_memory() 内で(のみ)使用されている.

(VMError::report_java_out_of_memory() は OutOfMemoryError が起こった際に呼び出される関数.
 なお, この関数が呼ばれてもその場で異常終了するわけではなく,
 この関数の呼び元では(呼び元は複数あるがどこでも), その後普通に OutOfMemoryError に対する例外ハンドリング処理が行われる.)

#### 参考(for your information): VMError::report_java_out_of_memory()
See: [here](no4230L4E.html) for details
### 内部構造(Internal structure)
行う処理は, コマンドラインオプションを取得して os::fork_and_exec() しているだけ.

#### 参考(for your information): VM_ReportJavaOutOfMemory::doit()
See: [here](no4230YCL.html) for details



### 詳細(Details)
See: [here](../doxygen/classVM__ReportJavaOutOfMemory.html) for details

---
