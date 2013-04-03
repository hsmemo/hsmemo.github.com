---
layout: default
title: Field に関する処理 ： 概要
---
[Up](noUsch6p8F.html) [Top](../index.html)

#### Field に関する処理 ： 概要

--- 
## 概要(Summary)
各クラスのフィールド情報は instanceKlass の fields フィールドに格納されている (See: instanceKlass).

ただし, 実行時に毎回 Constant Pool 情報を見にいくのは効率が悪いため, 
初回のアクセス時に ConstantPoolCache (CPCache) を生成し, 
ここからフィールド情報を取得している (See: ConstantPoolCache).
このため, 2回目以降は, CPCache から対応する情報を取ってきてアドレスを求め, そこにアクセスするだけですむ.


### static field について
static field は, 対応する mirror オブジェクト内に存在する.

各 static field の位置は (instance field と同じく) ClassFileParser::parseClassFile() の最後で決定される.
位置は instanceMirrorKlass::offset_of_static_fields() の後ろに配置される
(instanceMirrorKlass::offset_of_static_fields() 及び
 instanceKlass::low_offset/instanceKlass::high_offset を設定している箇所参照).

(なお, java.lang.Class がロードされる前にロードされたクラスについては, 
instanceMirrorKlass::offset_of_static_fields() が分からない.
このため, これらについては一旦保留にしておき, 
java_lang_Class::fixup_mirror() 内で改めて static field の位置が正式に設定されている)


## 備考(Notes)
volatile field については, 以下の re-order を防げばよい
(参考: <http://g.oswego.edu/dl/jmm/cookbook.html> JSR 133 Cookbook).

  1. volatile load より前への (normal load 及び normal store の) re-order
  2. volatile store より後ろへの (normal load 及び normal store の) re-order
  3. volatile load/store 間での re-order

"3." については, "1." "2." でメモリバリアを張れば
volatile load/store 間の re-order もかなり防止されるため, 
防ぐ必要があるのは volatile store を volatile load が追い抜くケースのみ.

(なお, これはあくまで Java レベルの話なので,
 VM 内部で管理構造の変更等を行う際には別途メモリバリア張って対処する必要がある)






