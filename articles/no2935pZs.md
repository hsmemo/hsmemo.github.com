---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スタックフレーム (Stack Frame) ： NotifyFramePop() の処理  
---
[Up](noa-wMJL5x.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スタックフレーム (Stack Frame) ： NotifyFramePop() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

NotifyFramePop() の処理は以下のように行われる.

1. まず, JvmtiEnv::NotifyFramePop() が呼ばれると最終的に JvmtiFramePops::set() が呼び出され, 
   JvmtiFramePops 内に指定された frame の番号が記録される.

2. 次に, メソッドが終了するたびに呼び出される JvmtiExport::post_method_exit() の中で, 
   JvmtiThreadState::cur_stack_depth() で現在のスタックフレームの深さを取得し
   JvmtiEnvThreadState::is_frame_pop() で通知対象のフレームかどうかが調べられる.

   対象のフレームであれば FramePop イベントが通知される
   (その後, 通知済みの frame 情報は JvmtiEnvThreadState::clear_frame_pop() で消去される).

## 備考(Notes)
JVMTI の PopFrame() 関数でポップされた場合は (JVMTI の仕様通り) FramePop イベントは発生させない (See: [here](no2935cDo.html) for details).

(JvmtiEnvThreadState::is_frame_pop() が true を返しても, そのまま JvmtiEnvThreadState::clear_frame_pop() で消去するだけ)

```
JvmtiEnv::PopFrame()
-> JvmtiThreadState::update_for_pop_top_frame()
   -> JvmtiThreadState::cur_stack_depth()
   -> JvmtiEnvThreadState::is_frame_pop()
   -> JvmtiEnvThreadState::clear_frame_pop()
```


```
    ((cite: hotspot/src/share/vm/prims/jvmtiEnv.cpp))
    jvmtiError
    JvmtiEnv::PopFrame(JavaThread* java_thread) {
    ...
        state->update_for_pop_top_frame();
```


```
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.cpp))
    void JvmtiThreadState::update_for_pop_top_frame() {
    ...
          for (JvmtiEnvThreadState* ets = it.first(); ets != NULL; ets = it.next(ets)) {
            if (ets->is_frame_pop(popframe_number)) {
              ets->clear_frame_pop(popframe_number);
```

## 備考(Notes)
FramePop イベントが有効になっている場合は, その通知契機を調べるため,
メソッドの終了時には毎回 JvmtiThreadState::cur_stack_depth() というメソッドで現在のスタックの深さを調べている.

しかし毎回この処理をまじめに行うと重いため, 一度調べた結果をキャッシュしており,
JvmtiExport::post_method_entry() や JvmtiExport::post_method_exit() 内でキャッシュ結果をインクリメント／デクリメントすることで計算処理の代わりとしている.

(See: JvmtiThreadState::invalidate_cur_stack_depth(),
      JvmtiThreadState::incr_cur_stack_depth(),
      JvmtiThreadState::decr_cur_stack_depth())

## 備考(Notes)
NotifyFramePop() によるイベント通知は interp_only_mode なイベント (See: [here](no3059eFS.html) for details).


## 処理の流れ (概要)(Execution Flows : Summary)
### JvmtiEnv::NotifyFramePop() の呼び出し時の処理
```
JvmtiEnv::NotifyFramePop()
-> JvmtiEnvThreadState::set_frame_pop()
   -> JvmtiEventController::set_frame_pop()
      -> JvmtiEventControllerPrivate::set_frame_pop()
         -> JvmtiFramePops::set()
```

### 各メソッドの終了時の処理
```
JvmtiExport::post_method_exit()
-> JvmtiThreadState::cur_stack_depth()
   -> JvmtiThreadState::count_frames()
-> JvmtiEnvThreadState::is_frame_pop()
   -> JvmtiFramePops::contains()
-> FramePop イベントの通知
-> JvmtiEnvThreadState::clear_frame_pop()
   -> JvmtiEventController::clear_frame_pop()
      -> JvmtiEventControllerPrivate::clear_frame_pop()
         -> JvmtiEventControllerPrivate::clear_frame_pop()
            -> JvmtiFramePops::clear()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::NotifyFramePop()
See: [here](no17119bqz.html) for details
### JvmtiEnvThreadState::set_frame_pop()
See: [here](no17119N0C.html) for details
### JvmtiEventController::set_frame_pop()
See: [here](no17119a-I.html) for details
### JvmtiEventControllerPrivate::set_frame_pop()
See: [here](no17119nIP.html) for details
### JvmtiFramePops::set()
See: [here](no171190SV.html) for details

### JvmtiExport::post_method_exit()
See: [here](no17119Onh.html) for details
### JvmtiThreadState::cur_stack_depth()
See: [here](no2935M2M.html) for details
### JvmtiThreadState::count_frames()
See: [here](no2935ZAT.html) for details
### JvmtiEnvThreadState::is_frame_pop()
See: [here](no17119bxn.html) for details
### JvmtiFramePops::contains()
See: [here](no17119o7t.html) for details
### JvmtiEnvThreadState::clear_frame_pop()
See: [here](no171191F0.html) for details
### JvmtiEventController::clear_frame_pop()
See: [here](no17119nPD.html) for details
### JvmtiEventControllerPrivate::clear_frame_pop()
See: [here](no171190ZJ.html) for details
### JvmtiFramePops::clear()
See: [here](no17119BkP.html) for details
### JvmtiThreadState::decr_cur_stack_depth()
See: [here](no2935_rG.html) for details






