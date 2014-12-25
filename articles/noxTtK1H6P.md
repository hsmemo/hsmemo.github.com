---
layout: default
title: Events クラス関連のクラス (Events, EventMark, 及びそれらの補助クラス(Event, EventBuffer))
---
[Top](../index.html)

#### Events クラス関連のクラス (Events, EventMark, 及びそれらの補助クラス(Event, EventBuffer))

これらは, デバッグ用(開発時用)のクラス.
より具体的に言うと, HotSpot 内で発生した事象のログを取るためのクラス.

### 概要(Summary)
以下のように使用する.

  * To log a single event use:

       Events::log("New nmethod has been created " INTPTR_FORMAT, nm);

  * To log a block of events use:

       EventMark m("GarbageCollecting %d", (intptr_t)gc_number);


```cpp
    ((cite: hotspot/src/share/vm/utilities/events.hpp))
    // Events and EventMark provide interfaces to log events taking place in the vm.
    // This facility is extremly useful for post-mortem debugging. The eventlog
    // often provides crucial information about events leading up to the crash.
    //
    // All arguments past the format string must be passed as an intptr_t.
    //
    // To log a single event use:
    //    Events::log("New nmethod has been created " INTPTR_FORMAT, nm);
    //
    // To log a block of events use:
    //    EventMark m("GarbageCollecting %d", (intptr_t)gc_number);
    //
    // The constructor to eventlog indents the eventlog until the
    // destructor has been executed.
    //
    // IMPLEMENTATION RESTRICTION:
    //   Max 3 arguments are saved for each logged event.
```



### クラス一覧(class list)

  * [Events](#noRh7UvCdH)
  * [EventMark](#nonT_8qTYv)
  * [Event](#novMP1zCD0)
  * [EventBuffer](#noNdb46U6L)


---
## <a name="noRh7UvCdH" id="noRh7UvCdH">Events</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時以外には空のクラスとして定義される).

デバッグ用のログの記録処理および出力処理を行うクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/share/vm/utilities/events.hpp))
    class Events : AllStatic {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
Events::log() でログを記録し, Events::print_all() または Events::print_last() で記録したログを出力する.

#### 情報の出力箇所(where the recorded information is output)
現在は, events() 関数および nevents() 関数内で(のみ)出力されている.

### 内部構造(Internal structure)
以下のようなメソッドを持つが,
開発時用のクラスであるため, #ifdef PRODUCT 時には全てのメソッドの中身が空になる.

(PRODUCT_RETURN は #ifdef PRODUCT 時には '{}' に展開される. (See: PRODUCT_RETURN))


```cpp
    ((cite: hotspot/src/share/vm/utilities/events.hpp))
      // Logs an event, format as printf
      static void log(const char* format, ...) PRODUCT_RETURN;
    
      // Prints all events in the buffer
      static void print_all(outputStream* st) PRODUCT_RETURN;
    
      // Prints last number events from the event buffer
      static void print_last(outputStream *st, int number) PRODUCT_RETURN;
```




### 詳細(Details)
See: [here](../doxygen/classEvents.html) for details

---
## <a name="nonT_8qTYv" id="nonT_8qTYv">EventMark</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時以外には空のクラスとして定義される).

ログの記録処理をソースコード上のスコープに合わせて行うための補助クラス(StackObjクラス)


```cpp
    ((cite: hotspot/src/share/vm/utilities/events.hpp))
    class EventMark : public StackObj {
```

### 使われ方(Usage)
コード中で EventMark 型の局所変数を宣言するだけ.

なお, 内部にはコンストラクタとデストラクタのみが定義されている.
さらに, 開発時用のクラスであるため, #ifdef PRODUCT 時には全ての中身が空になる.


```cpp
    ((cite: hotspot/src/share/vm/utilities/events.hpp))
      // log a begin event, format as printf
      EventMark(const char* format, ...) PRODUCT_RETURN;
      // log an end event
      ~EventMark() PRODUCT_RETURN;
```

### 内部構造(Internal structure)
コンストラクタで EventBuffer::add_event() を呼んで指定されたログを記録し,
デストラクタで (同じく EventBuffer::add_event() を呼んで) "done" というログを記録する.

(ついでに ログの indent レベルについても,
コンストラクタでインクリメントし, デストラクタでデクリメントしている)

ただし, このクラスは (デバッグ時であることに加えて) LogEvents オプションが指定されている場合にしか働かない.


```cpp
    ((cite: hotspot/src/share/vm/utilities/events.cpp))
    EventMark::EventMark(const char* format, ...) {
      if (LogEvents) {
        va_list ap;
        va_start(ap, format);
        intptr_t arg_1 = va_arg(ap, intptr_t);
        intptr_t arg_2 = va_arg(ap, intptr_t);
        intptr_t arg_3 = va_arg(ap, intptr_t);
        va_end(ap);
    
        EventBuffer::add_event(format, arg_1, arg_2, arg_3);
        EventBuffer::inc_indent();
      }
    }
    
    EventMark::~EventMark() {
      if (LogEvents) {
        EventBuffer::dec_indent();
        EventBuffer::add_event("done", 0, 0, 0);
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classEventMark.html) for details

---
## <a name="novMP1zCD0" id="novMP1zCD0">Event</a>

### 概要(Summary)
EventBuffer クラス内で使用される補助クラス.

1つの Event オブジェクトが 1つのログの内容に対応する.


```cpp
    ((cite: hotspot/src/share/vm/utilities/events.cpp))
    class Event VALUE_OBJ_CLASS_SPEC  {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
EventBuffer クラスの buffer フィールド (static フィールド) に(のみ)格納されている.

(正確には, このフィールドは Event の配列を格納するフィールド.
この中に, 使用される全ての Event オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
メモリ領域は EventBuffer::init() 内で(のみ)確保されている. 

そのメモリ領域中に個別の Event オブジェクトを書き込む作業は EventBuffer::add_event() 内で(のみ)行われている.




### 詳細(Details)
See: [here](../doxygen/classEvent.html) for details

---
## <a name="noNdb46U6L" id="noNdb46U6L">EventBuffer</a>

### 概要(Summary)
Events クラスおよび EventMark クラス内で使用される補助クラス.

実際のログの記録処理や出力処理はこのクラスに実装されている
(Events クラスや EventMark クラスはこのクラスのラッパーといった感じ).


```cpp
    ((cite: hotspot/src/share/vm/utilities/events.cpp))
    ////////////////////////////////////////////////////////////////////////////
    // EventBuffer
    //
    // Simple lock-free event queue. Every event has a unique 32-bit id.
    // It's fine if two threads add events at the same time, because they
    // will get different event id, and then write to different buffer location.
    // However, it is assumed that add_event() is quick enough (or buffer size
    // is big enough), so when one thread is adding event, there can't be more
    // than "size" events created by other threads; otherwise we'll end up having
    // two threads writing to the same location.
    
    class EventBuffer : AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classEventBuffer.html) for details

---
