---
layout: default
title: 処理時間計測用のユーティリティ・クラス (elapsedTimer, TimeStamp, TraceTime, TraceCPUTime)
---
[Top](../index.html)

#### 処理時間計測用のユーティリティ・クラス (elapsedTimer, TimeStamp, TraceTime, TraceCPUTime)

これらは, 処理時間を計測するためのユーティリティ・クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.hpp))
    // Timers for simple measurement.
```


### クラス一覧(class list)

  * [elapsedTimer](#noWhKe36vG)
  * [TimeStamp](#noq-kz9MyE)
  * [TraceTime](#noM8IuvRD9)
  * [TraceCPUTime](#noxr-ge4q3)


---
## <a name="noWhKe36vG" id="noWhKe36vG">elapsedTimer</a>

### 概要(Summary)
処理時間の簡単な計測処理を行うためのユーティリティ・クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.hpp))
    class elapsedTimer VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. elapsedTimer::start() で計測を開始し, elapsedTimer::stop() で計測を終了する.

2. 終了後は, elapsedTimer::ticks() や elapsedTimer::seconds(), elapsedTimer::milliseconds() で時間を取得できる
   (それぞれ計測時間を, OS の ticks 単位, 秒単位, ミリ秒単位, で返す).

2. なお, 計測中に (stop() させずに) それまでの時間を取得するための elapsedTimer::active_ticks() というメソッドもある.

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
現状の実装では os::elapsed_counter() で時間を取得している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.cpp))
    void elapsedTimer::start() {
      if (!_active) {
        _active = true;
        _start_counter = os::elapsed_counter();
      }
    }
    
    void elapsedTimer::stop() {
      if (_active) {
        _counter += os::elapsed_counter() - _start_counter;
        _active = false;
      }
    }
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.hpp))
      jlong ticks() const        { return _counter; }
```

```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.cpp))
    double elapsedTimer::seconds() const {
      double count = (double) _counter;
      double freq  = (double) os::elapsed_frequency();
      return count/freq;
    }
    
    jlong elapsedTimer::milliseconds() const {
      jlong ticks_per_ms = os::elapsed_frequency() / 1000;
      return _counter / ticks_per_ms;
    }
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.cpp))
    jlong elapsedTimer::active_ticks() const {
      if (!_active) {
        return ticks();
      }
      jlong counter = _counter + os::elapsed_counter() - _start_counter;
      return counter;
    }
```




### 詳細(Details)
See: [here](../doxygen/classelapsedTimer.html) for details

---
## <a name="noq-kz9MyE" id="noq-kz9MyE">TimeStamp</a>

### 概要(Summary)
処理時間の簡単な計測処理を行うためのユーティリティ・クラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.hpp))
    // TimeStamp is used for recording when an event took place.
    class TimeStamp VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. まず TimeStamp::update() または TimeStamp::update_to() で起点となる時間を設定する
   (update_to() では起点となる時間を明示的に指定する. update() では現在の時間が起点になる).

2. その後は, TimeStamp::ticks_since_update(), TimeStamp::seconds(), 
   および TimeStamp::milliseconds() で起点からの経過時間を取得できる
   (それぞれ経過時間を, OS の ticks 単位, 秒単位, ミリ秒単位, で返す).

3. 起点を変更したくなったら, また TimeStamp::update() や TimeStamp::update_to() を呼べばよい.

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
現状の実装では os::elapsed_counter() で時間を取得している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.cpp))
    void TimeStamp::update_to(jlong ticks) {
      _counter = ticks;
      if (_counter == 0)  _counter = 1;
      assert(is_updated(), "must not look clear");
    }
    
    void TimeStamp::update() {
      update_to(os::elapsed_counter());
    }
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.cpp))
    double TimeStamp::seconds() const {
      assert(is_updated(), "must not be clear");
      jlong new_count = os::elapsed_counter();
      double count = (double) new_count - _counter;
      double freq  = (double) os::elapsed_frequency();
      return count/freq;
    }
    
    jlong TimeStamp::milliseconds() const {
      assert(is_updated(), "must not be clear");
    
      jlong new_count = os::elapsed_counter();
      jlong count = new_count - _counter;
      jlong ticks_per_ms = os::elapsed_frequency() / 1000;
      return count / ticks_per_ms;
    }
    
    jlong TimeStamp::ticks_since_update() const {
      assert(is_updated(), "must not be clear");
      return os::elapsed_counter() - _counter;
    }
```




### 詳細(Details)
See: [here](../doxygen/classTimeStamp.html) for details

---
## <a name="noM8IuvRD9" id="noM8IuvRD9">TraceTime</a>

### 概要(Summary)
処理時間の簡単な計測処理を行うためのユーティリティ・クラス(StackObjクラス).

処理時間の計測処理をソースコード上のスコープに合わせて行うことができる
(= コンストラクタが呼ばれてからデストラクタが呼ばれるまでの時間を計測する).

計測結果は, 指定された elapsedTimer オブジェクトに足し込まれたり, 
指定のログファイル(デフォルトでは tty)に出力される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.hpp))
    // TraceTime is used for tracing the execution time of a block
    // Usage:
    //  { TraceTime t("block time")
    //    some_code();
    //  }
    //
    
    class TraceTime: public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で TraceTime 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
#### 定義されているフィールド
内部には elapsedTimer オブジェクトを保持している
(現状の実装では elapsedTimer を用いて時間を取得している).


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.hpp))
      elapsedTimer  _t;         // timer
```

#### 内部の処理
コンストラクタで elapsedTimer::start() を呼んで計測を開始する.

デストラクタで elapsedTimer::stop() で計測を終了する.
計測結果は, 以下の2つの用途に使用される.

* コンストラクタ引数で指定された elapsedTimer に足し込まれる.
* コンストラクタ引数で指定された outputStream (指定が無ければ tty) に出力される.

(なお, doit コンストラクタ引数が false だった場合(= _active フィールドが false になる場合)は, 
 このオブジェクトによる処理(計測処理／出力処理)は全て省略される)


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.cpp))
    TraceTime::TraceTime(const char* title,
                         bool doit,
                         bool print_cr,
                         outputStream* logfile) {
      _active   = doit;
      _verbose  = true;
      _print_cr = print_cr;
      _logfile = (logfile != NULL) ? logfile : tty;
    
      if (_active) {
        _accum = NULL;
        if (PrintGCTimeStamps) {
          _logfile->stamp();
          _logfile->print(": ");
        }
        _logfile->print("[%s", title);
        _logfile->flush();
        _t.start();
      }
    }
    
    TraceTime::TraceTime(const char* title,
                         elapsedTimer* accumulator,
                         bool doit,
                         bool verbose,
                         outputStream* logfile) {
      _active = doit;
      _verbose = verbose;
      _print_cr = true;
      _logfile = (logfile != NULL) ? logfile : tty;
      if (_active) {
        if (_verbose) {
          if (PrintGCTimeStamps) {
            _logfile->stamp();
            _logfile->print(": ");
          }
          _logfile->print("[%s", title);
          _logfile->flush();
        }
        _accum = accumulator;
        _t.start();
      }
    }
    
    TraceTime::~TraceTime() {
      if (_active) {
        _t.stop();
        if (_accum!=NULL) _accum->add(_t);
        if (_verbose) {
          if (_print_cr) {
            _logfile->print_cr(", %3.7f secs]", _t.seconds());
          } else {
            _logfile->print(", %3.7f secs]", _t.seconds());
          }
          _logfile->flush();
        }
      }
    }
```

### 備考(Notes)
なお, 一時的にタイマーを止めたり再会させたりすることもできる模様.


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.hpp))
      // Activation
      void suspend()  { if (_active) _t.stop();  }
      void resume()   { if (_active) _t.start(); }
```




### 詳細(Details)
See: [here](../doxygen/classTraceTime.html) for details

---
## <a name="noxr-ge4q3" id="noxr-ge4q3">TraceCPUTime</a>

### 概要(Summary)
処理時間の簡単な計測処理を行うためのユーティリティ・クラス(StackObjクラス).

処理時間の計測処理をソースコード上のスコープに合わせて行うことができる
(= コンストラクタが呼ばれてからデストラクタが呼ばれるまでの時間を計測する).

計測結果は, 指定されたログファイル(デフォルトでは tty)に出力される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.hpp))
    class TraceCPUTime: public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で TraceCPUTime 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

なお, 現在は PrintGCDetails オプションが指定されている場合にのみ有効になる模様
(全ての使用箇所で doit コンストラクタ引数として PrintGCDetails オプションの値が使用されているので).

### 内部構造(Internal structure)
コンストラクタで os::getTimesSecs() を呼んで計測を開始する.

デストラクタで os::getTimesSecs() で計測を終了し, 
コンストラクタ引数で指定された outputStream (指定が無ければ tty) に出力する.

(なお, doit コンストラクタ引数が false だった場合(= _active フィールドが false になる場合)は, 
 このオブジェクトによる処理(計測処理／出力処理)は全て省略される)


```cpp
    ((cite: hotspot/src/share/vm/runtime/timer.cpp))
    TraceCPUTime::TraceCPUTime(bool doit,
                   bool print_cr,
                   outputStream *logfile) :
      _active(doit),
      _print_cr(print_cr),
      _starting_user_time(0.0),
      _starting_system_time(0.0),
      _starting_real_time(0.0),
      _logfile(logfile),
      _error(false) {
      if (_active) {
        if (logfile != NULL) {
          _logfile = logfile;
        } else {
          _logfile = tty;
        }
    
        _error = !os::getTimesSecs(&_starting_real_time,
                                   &_starting_user_time,
                                   &_starting_system_time);
      }
    }
    
    TraceCPUTime::~TraceCPUTime() {
      if (_active) {
        bool valid = false;
        if (!_error) {
          double real_secs;                 // walk clock time
          double system_secs;               // system time
          double user_secs;                 // user time for all threads
    
          double real_time, user_time, system_time;
          valid = os::getTimesSecs(&real_time, &user_time, &system_time);
          if (valid) {
    
            user_secs = user_time - _starting_user_time;
            system_secs = system_time - _starting_system_time;
            real_secs = real_time - _starting_real_time;
    
            _logfile->print(" [Times: user=%3.2f sys=%3.2f, real=%3.2f secs] ",
              user_secs, system_secs, real_secs);
    
          } else {
            _logfile->print("[Invalid result in TraceCPUTime]");
          }
        } else {
          _logfile->print("[Error in TraceCPUTime]");
        }
         if (_print_cr) {
          _logfile->print_cr("");
        }
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classTraceCPUTime.html) for details

---
