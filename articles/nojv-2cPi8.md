---
layout: default
title: ErrorReporter クラス 
---
[Top](../index.html)

#### ErrorReporter クラス 



---
## <a name="no1BD6X4Ao" id="no1BD6X4Ao">ErrorReporter</a>

### 概要(Summary)
VMError クラス内で使用される補助クラス.


```cpp
    ((cite: hotspot/src/share/vm/utilities/errorReporter.hpp))
    class ErrorReporter : public StackObj {
```

エラーメッセージをログファイルに出力するためのクラス.

内部には, 以下のメソッド(のみ)が定義されている.


```cpp
    ((cite: hotspot/src/share/vm/utilities/errorReporter.hpp))
      void call(FILE* fd, char *buffer, int length);
```

### 使われ方(Usage)
VMError::report_and_die() 内で(のみ)使用されている.


```cpp
    ((cite: hotspot/src/share/vm/utilities/vmError.cpp))
    void VMError::report_and_die() {
    ...
        // Run error reporting to determine whether or not to report the crash.
        if (!transmit_report_done && should_report_bug(first_error->_id)) {
          transmit_report_done = true;
          FILE* hs_err = ::fdopen(log.fd(), "r");
          if (NULL != hs_err) {
            ErrorReporter er;
            er.call(hs_err, buffer, O_BUFLEN);
          }
        }
```

### 内部構造(Internal structure)
が, 肝心の ErrorReporter::call() の中身がないので, 現状では何も出力されない.


```cpp
    ((cite: hotspot/src/share/vm/utilities/errorReporter.cpp))
    void ErrorReporter::call(FILE* fd, char* buffer, int length) {
    }
```




### 詳細(Details)
See: [here](../doxygen/classErrorReporter.html) for details

---
