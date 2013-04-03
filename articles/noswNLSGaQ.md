---
layout: default
title: defaultStream クラス 
---
[Top](../index.html)

#### defaultStream クラス 



---
## <a name="now3RY5fQi" id="now3RY5fQi">defaultStream</a>

### 概要(Summary)
標準出力, 標準エラー出力, 
及びデフォルトのログファイル(hotspot.log)への出力を管理するクラス(outputStream クラス).


```
    ((cite: hotspot/src/share/vm/utilities/defaultStream.hpp))
    class defaultStream : public xmlTextStream {
```

ただし, ログファイル(hotspot.log)への出力機能は,
関連する diagnostic オプションが指定されている場合にのみ使用される (See: LogVMOutput, LogCompilation).

(LogVMOutput オプションまたは LogCompilation オプションが指定されている場合にしか初期化されない)


```
    ((cite: hotspot/src/share/vm/utilities/ostream.cpp))
    void defaultStream::init() {
      _inited = true;
      if (LogVMOutput || LogCompilation) {
        init_log();
      }
    }
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
defaultStream クラスの instance フィールド (static フィールド) に(のみ)格納されている.

(なお, 同じオブジェクトが tty 大域変数からも参照されている.
また, Xloggc オプションが指定されていない場合は, 同じオブジェクトが gclog_or_tty 大域変数からも参照されている.)

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

(tty->print(), tty->print_cr(), gclog_or_tty->print(), gclog_or_tty->print_cr(), 等の使用箇所を参照)

(また, 標準出力や標準エラー出力に対しては,
 defaultStream::error_stream() や defaultStream::output_stream() で FILE* を取得し,
 それを jio_fprintf() の引数にして出力処理を行うというパターンが結構多い)

(Unix 系の OS 用のソースコードでは,
 defaultStream::output_fd() や defaultStream::error_fd() で file descriptor を取得し,
 それを元に fdStream オブジェクトを作って出力するというパターンも多い)

### 内部構造(Internal structure)
標準出力と標準エラー出力は以下のフィールドに格納されている.

それぞれ defaultStream::error_stream() や defaultStream::output_stream() で対応する FILE* を取得できる.
また, defaultStream::output_fd() や defaultStream::error_fd() で対応する file descriptor を取得できる.


```
    ((cite: hotspot/src/share/vm/utilities/defaultStream.hpp))
      static int   _output_fd;
      static int   _error_fd;
      static FILE* _output_stream;
      static FILE* _error_stream;
```

(なお, これらはコマンドラインオプションの値によってどう対応するかが変わる)

* DisplayVMOutputToStderr オプションを指定していると, output として(標準出力ではなく)標準エラー出力が使われる.
* DisplayVMOutputToStdout オプションを指定していると, error として(標準エラー出力ではなく)標準出力が使われる.


```
    ((cite: hotspot/src/share/vm/utilities/defaultStream.hpp))
      static inline FILE* output_stream() {
        return DisplayVMOutputToStderr ? _error_stream : _output_stream;
      }
      static inline FILE* error_stream() {
        return DisplayVMOutputToStdout ? _output_stream : _error_stream;
      }
      static inline int output_fd() {
        return DisplayVMOutputToStderr ? _error_fd : _output_fd;
      }
      static inline int error_fd() {
        return DisplayVMOutputToStdout ? _output_fd : _error_fd;
      }
```

ログファイルは _outer_xmlStream フィールドに格納されている.


```
    ((cite: hotspot/src/share/vm/utilities/defaultStream.hpp))
      fileStream*  _log_file;  // XML-formatted file shared by all threads
```

### 備考(Notes)
ログファイル名は LogFile オプションで指定できる.

指定しなければデフォルトでは "hotspot.log" という名前が使われる.
さらに, "hotspot.log" が開けない場合は hs_pid${pid}.log というファイル名になる.


```
    ((cite: hotspot/src/share/vm/utilities/ostream.cpp))
    void defaultStream::init_log() {
      // %%% Need a MutexLocker?
      const char* log_name = LogFile != NULL ? LogFile : "hotspot.log";
      const char* try_name = make_log_name(log_name, NULL);
      fileStream* file = new(ResourceObj::C_HEAP) fileStream(try_name);
      if (!file->is_open()) {
        // Try again to open the file.
        char warnbuf[O_BUFLEN*2];
        jio_snprintf(warnbuf, sizeof(warnbuf),
                     "Warning:  Cannot open log file: %s\n", try_name);
        // Note:  This feature is for maintainer use only.  No need for L10N.
        jio_print(warnbuf);
        FREE_C_HEAP_ARRAY(char, try_name);
        try_name = make_log_name("hs_pid%p.log", os::get_temp_directory());
```




### 詳細(Details)
See: [here](../doxygen/classdefaultStream.html) for details

---
