---
layout: default
title: HotSpot の起動/終了処理 ： java コマンド (launcher) 部分の処理の流れ
---
[Up](noj08pougn.html) [Top](../index.html)

#### HotSpot の起動/終了処理 ： java コマンド (launcher) 部分の処理の流れ

--- 


## 概要(Summary)
(以下の内容はほとんど [HotSpot Runtime Overview](http://openjdk.java.net/groups/hotspot/docs/RuntimeOverview.html#VM%20Lifecycle|outline) の受け売り. こちらも参照のこと)

java コマンドによる実行の流れは以下のようになる
(ただし, 実際の HotSpot の起動処理／終了処理に当たる JNI_CreateJavaVM() と DestroyJavaVM() については別途説明 (See: [here](no2114J7x.html) and [here](no3059oro.html) for details)).

  1. コマンドラインオプションのパース

     (ただし, ここで処理されるのはいくつかのオプションだけ (e.g. -server, -client, etc).
     それ以外のオプションは JavaVMInitArgs で HotSpot に渡される(?))

  2. コマンドラインオプションでヒープサイズや JIT コンパイラ種別(client/server)が指定されていない場合には, 適当な値を設定しておく.

  3. LD_LIBRARY_PATH や CLASSPATH 等の環境変数を設定する.

     (<= LD_LIBRARY_PATH が適切に設定されてなかった場合には,
     設定した後で自分自身を execve() しなおすという荒技も...)

  4. もしコマンドラインで (クラス名ではなく) 実行可能jarファイルが指定されていた場合は, 
     jar ファイル中のマニフェストからメインクラス名を取得しておく.
  
  5. メインスレッドとして使用するスレッド(non-primordial thread)を作成する.
     その後, そのメインスレッドが JNI_CreateJavaVM() を呼ぶことで HotSpot を作成＆初期化する.
     
     (なお, "primordial thread" とは, 
     java コマンドを起動したときに最初に作られるスレッド (= デフォルトのメインスレッド) のこと)
     
     (わざわざ新しいスレッド(non-primordial thread)を作るのは,
     環境によっては primordial thread に色々な制限があり, 制御上やりにくい点が多数存在するため
     (例: スタックサイズやスタックの場所を設定しにくい, スタック上にガードページも配置しにくい, 等々)
     ([参考](<http://bugs.sun.com/view_bug.do?bug_id=6316197>)))

  6. HotSpot の生成と初期化が終了したら, メインクラスをロードし, その中から main() メソッドを取得する.

  7. CallStaticVoidMethod() で main() メソッドを呼び出す.

     (<= この際にコマンドラインオプションも marshalled されて渡されている)

  8. main メソッドが終了したら, pending になっている例外がないか確認する.

     (exit status を運んできているかもしれないため)

     例外は ExceptionOccurred で処理される
     (ExceptionOccurred の返り値が 0 なら正常, それ以外なら異常).
     この値は親プロセスに返される.

  9. DetachCurrentThread() を使ってメインスレッドがデタッチされる.

     (これによってスレッドカウントが減少し
     DestroyJavaVM() が安全に実行できるようになる)

     (また, そのスレッドが HotSpot 内で実行していないことやスタックにフレームが残っていないことも保証される)


## 処理の流れ (概要)(Execution Flows : Summary)
### 全体概要
実際の処理の流れは以下の通り.

  1. java コマンドを起動すると main() 関数 (Windows の場合は WinMain()) が実行される.
     ただし main() 内での処理は JLI_Launch() を呼び出すことだけ.

     (main() の中身が非常に単純なので 
      java コマンドを実現するソースファイルは 1つしかなかったりする... (jdk/src/share/bin/main.c))
     
     (<= なお, hotspot/src/share/tools/launcher/java.c は 
     gamma コマンドのソースファイルであって java コマンドのソースファイルではない.
     紛らわしいので注意)

  2. main() が呼び出す JLI_Launch() 内では以下の処理が行われる.

     1. LoadJavaVM() を呼び出して, libjvm をダイナミックロードする.

     2. 

     3. ParseArguments() を呼び出して, 
        いくつかのコマンドラインオプションをこの時点で処理してしまう (e.g. -classpath, -version, etc).
     
        (あるいは, -verify のような簡略型のオプションを -Xverify:all のような形に正規化する作業も行っている模様)

        (なお, それ以外のオプションは JavaVMInitArgs で HotSpot に渡される(?))

     4. 最後に ContinueInNewThread() を呼び出して, 
        メインメソッドを実行するための新しいスレッド("non-primordial thread")を立ち上げる.

        non-primordial thread のエントリポイントは JavaMain() になっており, この中で JNI_CreateJavaVM() が呼び出される.
        
        ("non-primordial thread" の役割については上述)

        (なお, 呼び出したスレッド自身は non-primordial thread スレッドの実行が終了するまでブロックされる)

  3. non-primordial thread が呼び出す JNI_CreateJavaVM() 内では以下の処理が行われる

     1. InitializeJVM() 内で JNI_CreateJavaVM() を呼び出し, HotSpot の起動と初期化を行う.

        (JNI_CreateJavaVM() は実際の HotSpot の起動処理に当たる JNI Invocation API (See: [here](no2114J7x.html) for details))

     2. LoadMainClass() を呼んで, メインクラスをロードする.

     3. JNI の GetStaticMethodID() 関数により, メインクラスの main() メソッドを取得する.

     4. JNI の CallStaticVoidMethod() 関数により, メインクラスの main() メソッドを呼び出す.

  4. 最終的に, メインクラスが正常終了した後で行われることは, 以下の通り

     1. JNI の DestroyJavaVM() 関数を呼び出して, HotSpot を終了させる.

        (DestroyJavaVM() は実際の HotSpot の終了処理に当たる JNI Invocation API (See: [here](no3059oro.html) for details))

### primordial thread による処理
```
main()
-> JLI_Launch()
   -> ...(#TODO)
   -> LoadJavaVM()
      -> OS によって処理が異なる
         * Solaris 又は Linux の場合:
           -> dlopen()          (← libjvm をロード)
           -> dlsym()           (← JNI_CreateJavaVM() を取得)
           -> dlsym()           (← JNI_GetDefaultJavaVMInitArgs() を取得)
         * Windows の場合:
           -> 
           -> LoadLibrary()     (← libjvm をロード)
           -> GetProcAddress()  (← JNI_CreateJavaVM() を取得)
           -> GetProcAddress()  (← JNI_GetDefaultJavaVMInitArgs() を取得)
   -> ...(#TODO)
   -> ContinueInNewThread()
      -> ContinueInNewThread0()
         -> OS によって処理が異なる.
            (どの場合も, 新しいスレッドを作成し, そのスレッドに残りの処理をやらせる.
             新規スレッドを作成した元々のメインスレッドは, 新規スレッドが終了するまでブロック)
            * Linux の場合:
              -> pthread_create()  (← なお, エントリポイントとしては JavaMain() 関数が指定されている)
              -> pthread_join()
            * Solaris の場合:
              -> thr_create()      (← なお, エントリポイントとしては JavaMain() 関数が指定されている)
              -> thr_join()
            * Windows の場合:
              -> _beginthredex()   (← なお, エントリポイントとしては JavaMain() 関数が指定されている)
              -> WaitForSingleObject()
```

### non-primordial thread による処理
```
JavaMain()
-> InitializeJVM()
   -> JNI_CreateJavaVM()
      -> (ここで HotSpot の起動処理が行われる) (See: [here](no2114J7x.html) for details)
-> ...#TODO
-> LoadMainClass()
   -> (See: [here](noSl0AuhYv.html) for details)
-> jni_GetStaticMethodID()
-> jni_CallStaticVoidMethod()
-> LEAVE() マクロ
   (最後まで特に問題が起きなければ LEAVE() に到達する)
    -> jni_DestroyJavaVM()
       -> (ここで HotSpot の終了処理が行われる) (See: [here](no3059oro.html) for details)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### main()
See: [here](no31977naC.html) for details
### WinMain()
See: [here](no31977BvO.html) for details
### JLI_Launch()
See: [here](no42309Jz.html) for details
### LoadJavaVM()  (Solaris の場合) (Linux の場合)
See: [here](no4230vTC.html) for details
(注: 定義ファイルは 'solaris/' ディレクトリ下にあるが Linux の場合にもこれが使用される)

(<= gamma の方では同名の関数が hotspot/src/os/posix/launcher/java_md.c で定義されているのに何故 java コマンドでは solaris...)

### LoadJavaVM()  (Windows の場合)
See: [here](no31977O5U.html) for details
### ContinueInNewThread()
See: [here](no17119spC.html) for details
### ContinueInNewThread0()  (Solaris の場合) (Linux の場合)
See: [here](no171195zI.html) for details
(注: 定義ファイルは 'solaris/' ディレクトリ下にあるが Linux の場合にもこれが使用される)

### ContinueInNewThread0()  (Windows の場合)
See: [here](no31977bDb.html) for details
### JavaMain()
See: [here](no42308dI.html) for details
### InitializeJVM()
See: [here](no319771Xn.html) for details
### LEAVE() マクロ
See: [here](no31977Cit.html) for details





