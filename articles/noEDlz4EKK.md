---
layout: default
title: ソースコードのビルド方法(How to Build)
---
[Up](no3yhpM-MW.html) [Top](../index.html)

#### ソースコードのビルド方法(How to Build)

--- 
(詳細は次のページを参考のこと.
 <http://hg.openjdk.java.net/jdk7/build/raw-file/tip/README-builds.html>)

0. 必要なライブラリ, ヘッダーファイル, gcc, ant などを揃える.

   有名なディストリビューション(Fedora, Ubuntu, etc)を使っているなら,
   <http://hg.openjdk.java.net/jdk7/jdk7/raw-file/tip/README-builds.html> の
   "Specific Developer Build Environments" を参照.

   例えば Ubuntu なら以下のコマンドで揃えられる.
   
   ```
   $ sudo aptitude build-dep openjdk-7
   ```

   (<= openjdk-6-jdk(javac) を別途入れる必要がある？)
   
   ```
   $ sudo aptitude install openjdk-6-jdk
   ```

   (なお, Freetype のファイルはオプショナルなのでなくてもよい.)

1. Mercurial をインストールする. (OpenJDK は Mercurial で管理されているため)

   (ちなみに, Mercurial を使わずに <http://download.java.net/openjdk/jdk7/> から直接ダウンロードしてくることもできる)

   ```
   $ sudo aptitude install mercurial
   ```

2. Mercurial の Forest extension をインストール. (OpenJDK は forest extension が提供する fclone コマンドで取得するため)

   (なお, なくても取得する方法はある(後述))
   
   ```
   $ YourHgForest=好きなパス
   $ hg clone https://bitbucket.org/pmezard/hgforest-crew/overview/ ${YourHgForest}
   ```

   そして, ~/.hgrc に以下の内容を追加.
   
   ```
   [extensions]
   forest = YourHgForestのパス/forest.py
   ```

3. 必要なファイルを取ってくる.

   1. OpenJDk のソース
      
      ```
      $ hg fclone http://hg.openjdk.java.net/jdk7/jdk7/ jdk7
      ```

      なお, forest extension を使いたくなければ, 以下のようにしても良い.

      ```
      $ YourOpenJDKDir=好きなパス
      $ hg clone http://hg.openjdk.java.net/jdk7/jdk7 ${YourOpenJDKDir}
      $ cd ${YourOpenJDKDir}
      $ sh ./get_source.sh
      ```

   2. OpenJDK をコンパイルするための JDK (バイナリ)

      (JDK に含まれる標準ライブラリ(.java ファイル)をコンパイルする際に, 作ったばかりの javac を使うのは不安な人のため)

      JDK 6u18 以上が必要. 適当な JDK (OpenJDK6, Sun JDK, etc) をインストールする.

4. ビルド用の環境変数を設定する

   ```
   $ export ALT_BOOTDIR=${OpenJDK をコンパイルするための JDK のディレクトリ}
   ```

   (例: export ALT_BOOTDIR=/usr/lib/jvm/java-6-openjdk)

   なお, 一度コンパイルしたことがあって, 一部のコンパイルだけをやり直したいというときには,
   ALT_JDK_IMPORT_PATH で前回の結果を指せばよい.

5. ビルドする

   OpenJDk のソースのディレクトリに移動し, 以下を実行すればいい

   ```
   $ . jdk/make/jdk_generic_profile.sh
   $ make sanity && make
   ```

   (jdk/make/jdk_generic_profile.sh で, ビルドに必要な環境変数のset/unset を行う.
    make sanity で検査を行い、問題ないなら make で実際にコンパイルする)

   (なお, make は GNU make でないと動かないので,
   solaris 等では make の代わりに gmake コマンドを使うこと.
   <http://hg.openjdk.java.net/jdk7/jdk7/raw-file/tip/README-builds.html>)

6. ビルド結果を確認

   (ビルド結果は /build/${ARCH} (i.e. /build/linux-amd64, etc) 以下に配置される)


## 備考(Notes)
そのままだと product 版しかビルドされない.

jvmg 版や fastdebug 版などを作るには, 以下のように環境変数をセットする.

```
export SKIP_DEBUG_BUILD=false       # jvmg の場合
export SKIP_FASTDEBUG_BUILD=false   # fastdebug の場合
```

## その他の Tips
### コンパイルオプションを独自に変更するには
adlc 用なら, hotspot/build/linux/makefiles/adlc.make を変更.

それ以外なら, hotspot/build/linux/makefiles/gcc.make を変更.

### jaxp のソースがないのでエラーと言われたら
ヒントに従い, ant に以下のどちらかのコマンドオプションを付ければよい.

```
ant -Dallow.downloads=true
ant -Ddrops.dir=${some_directory}
```

簡単なのは, make する前に以下の環境変数を export する方法.

```
export ALLOW_DOWNLOADS=true
make
```

あるいは手動で落としてきて ALT_DROPS_DIR に設定しても良い.

### OS の kernel version が対応していないバージョンだと言われたら
以下のように環境変数をセットすると回避できる.

```
export DISABLE_HOTSPOT_OS_VERSION_CHECK=ok
```

(See: <http://mail.openjdk.java.net/pipermail/porters-dev/2011-October/000366.html>)

### asound lib がないと言われたら
(/usr/lib 以下になくなったのが問題らしい)

以下のように環境変数をセットすると回避できる.

```
export EXTRA_LIBS=/usr/lib/x86_64-linux-gnu/libasound.so.2
```

(See: <http://mail.openjdk.java.net/pipermail/porters-dev/2011-October/000366.html>)







