---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/bytecodes.cpp

### 名前(function name)
```
void Bytecodes::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 既に初期化済みなら, ここでリターン
      ---------------------------------------- -}

	  if (_is_initialized) return;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(number_of_codes <= 256, "too many bytecodes");
	
  {- -------------------------------------------
  (1) 以下, Bytecodes::def() を使ってバイトコードテーブルを初期化する
      (わざわざ動的に作っているのは, 実行時に consistency check (sanity check 的なこと) を行いたいのと, 
       動的にやれば初期化処理の順番は bytecode のコード番号とは無関係に記述できるため, とのこと.)
       
      (なお, format が NULL であれば, その形式では使用されないという意味.
       また, format 文字列の意味に付いては Bytecodes::compute_flags() のコメント参照.
       See: Bytecodes::compute_flags())
  
      (実行後の TOS の型が不定なものについては result tp は T_ILLEGAL にしている)    
      ---------------------------------------- -}

	  // initialize bytecode tables - didn't use static array initializers
	  // (such as {}) so we can do additional consistency checks and init-
	  // code is independent of actual bytecode numbering.
	  //
	  // Note 1: NULL for the format string means the bytecode doesn't exist
	  //         in that form.
	  //
	  // Note 2: The result type is T_ILLEGAL for bytecodes where the top of stack
	  //         type after execution is not only determined by the bytecode itself.
	
	  //  Java bytecodes
	  //  bytecode               bytecode name           format   wide f.   result tp  stk traps
	  def(_nop                 , "nop"                 , "b"    , NULL    , T_VOID   ,  0, false);
	  def(_aconst_null         , "aconst_null"         , "b"    , NULL    , T_OBJECT ,  1, false);
	  def(_iconst_m1           , "iconst_m1"           , "b"    , NULL    , T_INT    ,  1, false);
	  def(_iconst_0            , "iconst_0"            , "b"    , NULL    , T_INT    ,  1, false);
	  def(_iconst_1            , "iconst_1"            , "b"    , NULL    , T_INT    ,  1, false);
	  def(_iconst_2            , "iconst_2"            , "b"    , NULL    , T_INT    ,  1, false);
	  def(_iconst_3            , "iconst_3"            , "b"    , NULL    , T_INT    ,  1, false);
	  def(_iconst_4            , "iconst_4"            , "b"    , NULL    , T_INT    ,  1, false);
	  def(_iconst_5            , "iconst_5"            , "b"    , NULL    , T_INT    ,  1, false);
	  def(_lconst_0            , "lconst_0"            , "b"    , NULL    , T_LONG   ,  2, false);
	  def(_lconst_1            , "lconst_1"            , "b"    , NULL    , T_LONG   ,  2, false);
	  def(_fconst_0            , "fconst_0"            , "b"    , NULL    , T_FLOAT  ,  1, false);
	  def(_fconst_1            , "fconst_1"            , "b"    , NULL    , T_FLOAT  ,  1, false);
	  def(_fconst_2            , "fconst_2"            , "b"    , NULL    , T_FLOAT  ,  1, false);
	  def(_dconst_0            , "dconst_0"            , "b"    , NULL    , T_DOUBLE ,  2, false);
	  def(_dconst_1            , "dconst_1"            , "b"    , NULL    , T_DOUBLE ,  2, false);
	  def(_bipush              , "bipush"              , "bc"   , NULL    , T_INT    ,  1, false);
	  def(_sipush              , "sipush"              , "bcc"  , NULL    , T_INT    ,  1, false);
	  def(_ldc                 , "ldc"                 , "bk"   , NULL    , T_ILLEGAL,  1, true );
	  def(_ldc_w               , "ldc_w"               , "bkk"  , NULL    , T_ILLEGAL,  1, true );
	  def(_ldc2_w              , "ldc2_w"              , "bkk"  , NULL    , T_ILLEGAL,  2, true );
	  def(_iload               , "iload"               , "bi"   , "wbii"  , T_INT    ,  1, false);
	  def(_lload               , "lload"               , "bi"   , "wbii"  , T_LONG   ,  2, false);
	  def(_fload               , "fload"               , "bi"   , "wbii"  , T_FLOAT  ,  1, false);
	  def(_dload               , "dload"               , "bi"   , "wbii"  , T_DOUBLE ,  2, false);
	  def(_aload               , "aload"               , "bi"   , "wbii"  , T_OBJECT ,  1, false);
	  def(_iload_0             , "iload_0"             , "b"    , NULL    , T_INT    ,  1, false);
	  def(_iload_1             , "iload_1"             , "b"    , NULL    , T_INT    ,  1, false);
	  def(_iload_2             , "iload_2"             , "b"    , NULL    , T_INT    ,  1, false);
	  def(_iload_3             , "iload_3"             , "b"    , NULL    , T_INT    ,  1, false);
	  def(_lload_0             , "lload_0"             , "b"    , NULL    , T_LONG   ,  2, false);
	  def(_lload_1             , "lload_1"             , "b"    , NULL    , T_LONG   ,  2, false);
	  def(_lload_2             , "lload_2"             , "b"    , NULL    , T_LONG   ,  2, false);
	  def(_lload_3             , "lload_3"             , "b"    , NULL    , T_LONG   ,  2, false);
	  def(_fload_0             , "fload_0"             , "b"    , NULL    , T_FLOAT  ,  1, false);
	  def(_fload_1             , "fload_1"             , "b"    , NULL    , T_FLOAT  ,  1, false);
	  def(_fload_2             , "fload_2"             , "b"    , NULL    , T_FLOAT  ,  1, false);
	  def(_fload_3             , "fload_3"             , "b"    , NULL    , T_FLOAT  ,  1, false);
	  def(_dload_0             , "dload_0"             , "b"    , NULL    , T_DOUBLE ,  2, false);
	  def(_dload_1             , "dload_1"             , "b"    , NULL    , T_DOUBLE ,  2, false);
	  def(_dload_2             , "dload_2"             , "b"    , NULL    , T_DOUBLE ,  2, false);
	  def(_dload_3             , "dload_3"             , "b"    , NULL    , T_DOUBLE ,  2, false);
	  def(_aload_0             , "aload_0"             , "b"    , NULL    , T_OBJECT ,  1, true ); // rewriting in interpreter
	  def(_aload_1             , "aload_1"             , "b"    , NULL    , T_OBJECT ,  1, false);
	  def(_aload_2             , "aload_2"             , "b"    , NULL    , T_OBJECT ,  1, false);
	  def(_aload_3             , "aload_3"             , "b"    , NULL    , T_OBJECT ,  1, false);
	  def(_iaload              , "iaload"              , "b"    , NULL    , T_INT    , -1, true );
	  def(_laload              , "laload"              , "b"    , NULL    , T_LONG   ,  0, true );
	  def(_faload              , "faload"              , "b"    , NULL    , T_FLOAT  , -1, true );
	  def(_daload              , "daload"              , "b"    , NULL    , T_DOUBLE ,  0, true );
	  def(_aaload              , "aaload"              , "b"    , NULL    , T_OBJECT , -1, true );
	  def(_baload              , "baload"              , "b"    , NULL    , T_INT    , -1, true );
	  def(_caload              , "caload"              , "b"    , NULL    , T_INT    , -1, true );
	  def(_saload              , "saload"              , "b"    , NULL    , T_INT    , -1, true );
	  def(_istore              , "istore"              , "bi"   , "wbii"  , T_VOID   , -1, false);
	  def(_lstore              , "lstore"              , "bi"   , "wbii"  , T_VOID   , -2, false);
	  def(_fstore              , "fstore"              , "bi"   , "wbii"  , T_VOID   , -1, false);
	  def(_dstore              , "dstore"              , "bi"   , "wbii"  , T_VOID   , -2, false);
	  def(_astore              , "astore"              , "bi"   , "wbii"  , T_VOID   , -1, false);
	  def(_istore_0            , "istore_0"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_istore_1            , "istore_1"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_istore_2            , "istore_2"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_istore_3            , "istore_3"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_lstore_0            , "lstore_0"            , "b"    , NULL    , T_VOID   , -2, false);
	  def(_lstore_1            , "lstore_1"            , "b"    , NULL    , T_VOID   , -2, false);
	  def(_lstore_2            , "lstore_2"            , "b"    , NULL    , T_VOID   , -2, false);
	  def(_lstore_3            , "lstore_3"            , "b"    , NULL    , T_VOID   , -2, false);
	  def(_fstore_0            , "fstore_0"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_fstore_1            , "fstore_1"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_fstore_2            , "fstore_2"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_fstore_3            , "fstore_3"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_dstore_0            , "dstore_0"            , "b"    , NULL    , T_VOID   , -2, false);
	  def(_dstore_1            , "dstore_1"            , "b"    , NULL    , T_VOID   , -2, false);
	  def(_dstore_2            , "dstore_2"            , "b"    , NULL    , T_VOID   , -2, false);
	  def(_dstore_3            , "dstore_3"            , "b"    , NULL    , T_VOID   , -2, false);
	  def(_astore_0            , "astore_0"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_astore_1            , "astore_1"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_astore_2            , "astore_2"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_astore_3            , "astore_3"            , "b"    , NULL    , T_VOID   , -1, false);
	  def(_iastore             , "iastore"             , "b"    , NULL    , T_VOID   , -3, true );
	  def(_lastore             , "lastore"             , "b"    , NULL    , T_VOID   , -4, true );
	  def(_fastore             , "fastore"             , "b"    , NULL    , T_VOID   , -3, true );
	  def(_dastore             , "dastore"             , "b"    , NULL    , T_VOID   , -4, true );
	  def(_aastore             , "aastore"             , "b"    , NULL    , T_VOID   , -3, true );
	  def(_bastore             , "bastore"             , "b"    , NULL    , T_VOID   , -3, true );
	  def(_castore             , "castore"             , "b"    , NULL    , T_VOID   , -3, true );
	  def(_sastore             , "sastore"             , "b"    , NULL    , T_VOID   , -3, true );
	  def(_pop                 , "pop"                 , "b"    , NULL    , T_VOID   , -1, false);
	  def(_pop2                , "pop2"                , "b"    , NULL    , T_VOID   , -2, false);
	  def(_dup                 , "dup"                 , "b"    , NULL    , T_VOID   ,  1, false);
	  def(_dup_x1              , "dup_x1"              , "b"    , NULL    , T_VOID   ,  1, false);
	  def(_dup_x2              , "dup_x2"              , "b"    , NULL    , T_VOID   ,  1, false);
	  def(_dup2                , "dup2"                , "b"    , NULL    , T_VOID   ,  2, false);
	  def(_dup2_x1             , "dup2_x1"             , "b"    , NULL    , T_VOID   ,  2, false);
	  def(_dup2_x2             , "dup2_x2"             , "b"    , NULL    , T_VOID   ,  2, false);
	  def(_swap                , "swap"                , "b"    , NULL    , T_VOID   ,  0, false);
	  def(_iadd                , "iadd"                , "b"    , NULL    , T_INT    , -1, false);
	  def(_ladd                , "ladd"                , "b"    , NULL    , T_LONG   , -2, false);
	  def(_fadd                , "fadd"                , "b"    , NULL    , T_FLOAT  , -1, false);
	  def(_dadd                , "dadd"                , "b"    , NULL    , T_DOUBLE , -2, false);
	  def(_isub                , "isub"                , "b"    , NULL    , T_INT    , -1, false);
	  def(_lsub                , "lsub"                , "b"    , NULL    , T_LONG   , -2, false);
	  def(_fsub                , "fsub"                , "b"    , NULL    , T_FLOAT  , -1, false);
	  def(_dsub                , "dsub"                , "b"    , NULL    , T_DOUBLE , -2, false);
	  def(_imul                , "imul"                , "b"    , NULL    , T_INT    , -1, false);
	  def(_lmul                , "lmul"                , "b"    , NULL    , T_LONG   , -2, false);
	  def(_fmul                , "fmul"                , "b"    , NULL    , T_FLOAT  , -1, false);
	  def(_dmul                , "dmul"                , "b"    , NULL    , T_DOUBLE , -2, false);
	  def(_idiv                , "idiv"                , "b"    , NULL    , T_INT    , -1, true );
	  def(_ldiv                , "ldiv"                , "b"    , NULL    , T_LONG   , -2, true );
	  def(_fdiv                , "fdiv"                , "b"    , NULL    , T_FLOAT  , -1, false);
	  def(_ddiv                , "ddiv"                , "b"    , NULL    , T_DOUBLE , -2, false);
	  def(_irem                , "irem"                , "b"    , NULL    , T_INT    , -1, true );
	  def(_lrem                , "lrem"                , "b"    , NULL    , T_LONG   , -2, true );
	  def(_frem                , "frem"                , "b"    , NULL    , T_FLOAT  , -1, false);
	  def(_drem                , "drem"                , "b"    , NULL    , T_DOUBLE , -2, false);
	  def(_ineg                , "ineg"                , "b"    , NULL    , T_INT    ,  0, false);
	  def(_lneg                , "lneg"                , "b"    , NULL    , T_LONG   ,  0, false);
	  def(_fneg                , "fneg"                , "b"    , NULL    , T_FLOAT  ,  0, false);
	  def(_dneg                , "dneg"                , "b"    , NULL    , T_DOUBLE ,  0, false);
	  def(_ishl                , "ishl"                , "b"    , NULL    , T_INT    , -1, false);
	  def(_lshl                , "lshl"                , "b"    , NULL    , T_LONG   , -1, false);
	  def(_ishr                , "ishr"                , "b"    , NULL    , T_INT    , -1, false);
	  def(_lshr                , "lshr"                , "b"    , NULL    , T_LONG   , -1, false);
	  def(_iushr               , "iushr"               , "b"    , NULL    , T_INT    , -1, false);
	  def(_lushr               , "lushr"               , "b"    , NULL    , T_LONG   , -1, false);
	  def(_iand                , "iand"                , "b"    , NULL    , T_INT    , -1, false);
	  def(_land                , "land"                , "b"    , NULL    , T_LONG   , -2, false);
	  def(_ior                 , "ior"                 , "b"    , NULL    , T_INT    , -1, false);
	  def(_lor                 , "lor"                 , "b"    , NULL    , T_LONG   , -2, false);
	  def(_ixor                , "ixor"                , "b"    , NULL    , T_INT    , -1, false);
	  def(_lxor                , "lxor"                , "b"    , NULL    , T_LONG   , -2, false);
	  def(_iinc                , "iinc"                , "bic"  , "wbiicc", T_VOID   ,  0, false);
	  def(_i2l                 , "i2l"                 , "b"    , NULL    , T_LONG   ,  1, false);
	  def(_i2f                 , "i2f"                 , "b"    , NULL    , T_FLOAT  ,  0, false);
	  def(_i2d                 , "i2d"                 , "b"    , NULL    , T_DOUBLE ,  1, false);
	  def(_l2i                 , "l2i"                 , "b"    , NULL    , T_INT    , -1, false);
	  def(_l2f                 , "l2f"                 , "b"    , NULL    , T_FLOAT  , -1, false);
	  def(_l2d                 , "l2d"                 , "b"    , NULL    , T_DOUBLE ,  0, false);
	  def(_f2i                 , "f2i"                 , "b"    , NULL    , T_INT    ,  0, false);
	  def(_f2l                 , "f2l"                 , "b"    , NULL    , T_LONG   ,  1, false);
	  def(_f2d                 , "f2d"                 , "b"    , NULL    , T_DOUBLE ,  1, false);
	  def(_d2i                 , "d2i"                 , "b"    , NULL    , T_INT    , -1, false);
	  def(_d2l                 , "d2l"                 , "b"    , NULL    , T_LONG   ,  0, false);
	  def(_d2f                 , "d2f"                 , "b"    , NULL    , T_FLOAT  , -1, false);
	  def(_i2b                 , "i2b"                 , "b"    , NULL    , T_BYTE   ,  0, false);
	  def(_i2c                 , "i2c"                 , "b"    , NULL    , T_CHAR   ,  0, false);
	  def(_i2s                 , "i2s"                 , "b"    , NULL    , T_SHORT  ,  0, false);
	  def(_lcmp                , "lcmp"                , "b"    , NULL    , T_VOID   , -3, false);
	  def(_fcmpl               , "fcmpl"               , "b"    , NULL    , T_VOID   , -1, false);
	  def(_fcmpg               , "fcmpg"               , "b"    , NULL    , T_VOID   , -1, false);
	  def(_dcmpl               , "dcmpl"               , "b"    , NULL    , T_VOID   , -3, false);
	  def(_dcmpg               , "dcmpg"               , "b"    , NULL    , T_VOID   , -3, false);
	  def(_ifeq                , "ifeq"                , "boo"  , NULL    , T_VOID   , -1, false);
	  def(_ifne                , "ifne"                , "boo"  , NULL    , T_VOID   , -1, false);
	  def(_iflt                , "iflt"                , "boo"  , NULL    , T_VOID   , -1, false);
	  def(_ifge                , "ifge"                , "boo"  , NULL    , T_VOID   , -1, false);
	  def(_ifgt                , "ifgt"                , "boo"  , NULL    , T_VOID   , -1, false);
	  def(_ifle                , "ifle"                , "boo"  , NULL    , T_VOID   , -1, false);
	  def(_if_icmpeq           , "if_icmpeq"           , "boo"  , NULL    , T_VOID   , -2, false);
	  def(_if_icmpne           , "if_icmpne"           , "boo"  , NULL    , T_VOID   , -2, false);
	  def(_if_icmplt           , "if_icmplt"           , "boo"  , NULL    , T_VOID   , -2, false);
	  def(_if_icmpge           , "if_icmpge"           , "boo"  , NULL    , T_VOID   , -2, false);
	  def(_if_icmpgt           , "if_icmpgt"           , "boo"  , NULL    , T_VOID   , -2, false);
	  def(_if_icmple           , "if_icmple"           , "boo"  , NULL    , T_VOID   , -2, false);
	  def(_if_acmpeq           , "if_acmpeq"           , "boo"  , NULL    , T_VOID   , -2, false);
	  def(_if_acmpne           , "if_acmpne"           , "boo"  , NULL    , T_VOID   , -2, false);
	  def(_goto                , "goto"                , "boo"  , NULL    , T_VOID   ,  0, false);
	  def(_jsr                 , "jsr"                 , "boo"  , NULL    , T_INT    ,  0, false);
	  def(_ret                 , "ret"                 , "bi"   , "wbii"  , T_VOID   ,  0, false);
	  def(_tableswitch         , "tableswitch"         , ""     , NULL    , T_VOID   , -1, false); // may have backward branches
	  def(_lookupswitch        , "lookupswitch"        , ""     , NULL    , T_VOID   , -1, false); // rewriting in interpreter
	  def(_ireturn             , "ireturn"             , "b"    , NULL    , T_INT    , -1, true);
	  def(_lreturn             , "lreturn"             , "b"    , NULL    , T_LONG   , -2, true);
	  def(_freturn             , "freturn"             , "b"    , NULL    , T_FLOAT  , -1, true);
	  def(_dreturn             , "dreturn"             , "b"    , NULL    , T_DOUBLE , -2, true);
	  def(_areturn             , "areturn"             , "b"    , NULL    , T_OBJECT , -1, true);
	  def(_return              , "return"              , "b"    , NULL    , T_VOID   ,  0, true);
	  def(_getstatic           , "getstatic"           , "bJJ"  , NULL    , T_ILLEGAL,  1, true );
	  def(_putstatic           , "putstatic"           , "bJJ"  , NULL    , T_ILLEGAL, -1, true );
	  def(_getfield            , "getfield"            , "bJJ"  , NULL    , T_ILLEGAL,  0, true );
	  def(_putfield            , "putfield"            , "bJJ"  , NULL    , T_ILLEGAL, -2, true );
	  def(_invokevirtual       , "invokevirtual"       , "bJJ"  , NULL    , T_ILLEGAL, -1, true);
	  def(_invokespecial       , "invokespecial"       , "bJJ"  , NULL    , T_ILLEGAL, -1, true);
	  def(_invokestatic        , "invokestatic"        , "bJJ"  , NULL    , T_ILLEGAL,  0, true);
	  def(_invokeinterface     , "invokeinterface"     , "bJJ__", NULL    , T_ILLEGAL, -1, true);
	  def(_invokedynamic       , "invokedynamic"       , "bJJJJ", NULL    , T_ILLEGAL,  0, true );
	  def(_new                 , "new"                 , "bkk"  , NULL    , T_OBJECT ,  1, true );
	  def(_newarray            , "newarray"            , "bc"   , NULL    , T_OBJECT ,  0, true );
	  def(_anewarray           , "anewarray"           , "bkk"  , NULL    , T_OBJECT ,  0, true );
	  def(_arraylength         , "arraylength"         , "b"    , NULL    , T_VOID   ,  0, true );
	  def(_athrow              , "athrow"              , "b"    , NULL    , T_VOID   , -1, true );
	  def(_checkcast           , "checkcast"           , "bkk"  , NULL    , T_OBJECT ,  0, true );
	  def(_instanceof          , "instanceof"          , "bkk"  , NULL    , T_INT    ,  0, true );
	  def(_monitorenter        , "monitorenter"        , "b"    , NULL    , T_VOID   , -1, true );
	  def(_monitorexit         , "monitorexit"         , "b"    , NULL    , T_VOID   , -1, true );
	  def(_wide                , "wide"                , ""     , NULL    , T_VOID   ,  0, false);
	  def(_multianewarray      , "multianewarray"      , "bkkc" , NULL    , T_OBJECT ,  1, true );
	  def(_ifnull              , "ifnull"              , "boo"  , NULL    , T_VOID   , -1, false);
	  def(_ifnonnull           , "ifnonnull"           , "boo"  , NULL    , T_VOID   , -1, false);
	  def(_goto_w              , "goto_w"              , "boooo", NULL    , T_VOID   ,  0, false);
	  def(_jsr_w               , "jsr_w"               , "boooo", NULL    , T_INT    ,  0, false);
	  def(_breakpoint          , "breakpoint"          , ""     , NULL    , T_VOID   ,  0, true);
	
	  //  JVM bytecodes
	  //  bytecode               bytecode name           format   wide f.   result tp  stk traps  std code
	
	  def(_fast_agetfield      , "fast_agetfield"      , "bJJ"  , NULL    , T_OBJECT ,  0, true , _getfield       );
	  def(_fast_bgetfield      , "fast_bgetfield"      , "bJJ"  , NULL    , T_INT    ,  0, true , _getfield       );
	  def(_fast_cgetfield      , "fast_cgetfield"      , "bJJ"  , NULL    , T_CHAR   ,  0, true , _getfield       );
	  def(_fast_dgetfield      , "fast_dgetfield"      , "bJJ"  , NULL    , T_DOUBLE ,  0, true , _getfield       );
	  def(_fast_fgetfield      , "fast_fgetfield"      , "bJJ"  , NULL    , T_FLOAT  ,  0, true , _getfield       );
	  def(_fast_igetfield      , "fast_igetfield"      , "bJJ"  , NULL    , T_INT    ,  0, true , _getfield       );
	  def(_fast_lgetfield      , "fast_lgetfield"      , "bJJ"  , NULL    , T_LONG   ,  0, true , _getfield       );
	  def(_fast_sgetfield      , "fast_sgetfield"      , "bJJ"  , NULL    , T_SHORT  ,  0, true , _getfield       );
	
	  def(_fast_aputfield      , "fast_aputfield"      , "bJJ"  , NULL    , T_OBJECT ,  0, true , _putfield       );
	  def(_fast_bputfield      , "fast_bputfield"      , "bJJ"  , NULL    , T_INT    ,  0, true , _putfield       );
	  def(_fast_cputfield      , "fast_cputfield"      , "bJJ"  , NULL    , T_CHAR   ,  0, true , _putfield       );
	  def(_fast_dputfield      , "fast_dputfield"      , "bJJ"  , NULL    , T_DOUBLE ,  0, true , _putfield       );
	  def(_fast_fputfield      , "fast_fputfield"      , "bJJ"  , NULL    , T_FLOAT  ,  0, true , _putfield       );
	  def(_fast_iputfield      , "fast_iputfield"      , "bJJ"  , NULL    , T_INT    ,  0, true , _putfield       );
	  def(_fast_lputfield      , "fast_lputfield"      , "bJJ"  , NULL    , T_LONG   ,  0, true , _putfield       );
	  def(_fast_sputfield      , "fast_sputfield"      , "bJJ"  , NULL    , T_SHORT  ,  0, true , _putfield       );
	
	  def(_fast_aload_0        , "fast_aload_0"        , "b"    , NULL    , T_OBJECT ,  1, true , _aload_0        );
	  def(_fast_iaccess_0      , "fast_iaccess_0"      , "b_JJ" , NULL    , T_INT    ,  1, true , _aload_0        );
	  def(_fast_aaccess_0      , "fast_aaccess_0"      , "b_JJ" , NULL    , T_OBJECT ,  1, true , _aload_0        );
	  def(_fast_faccess_0      , "fast_faccess_0"      , "b_JJ" , NULL    , T_OBJECT ,  1, true , _aload_0        );
	
	  def(_fast_iload          , "fast_iload"          , "bi"   , NULL    , T_INT    ,  1, false, _iload);
	  def(_fast_iload2         , "fast_iload2"         , "bi_i" , NULL    , T_INT    ,  2, false, _iload);
	  def(_fast_icaload        , "fast_icaload"        , "bi_"  , NULL    , T_INT    ,  0, false, _iload);
	
	  // Faster method invocation.
	  def(_fast_invokevfinal   , "fast_invokevfinal"   , "bJJ"  , NULL    , T_ILLEGAL, -1, true, _invokevirtual   );
	
	  def(_fast_linearswitch   , "fast_linearswitch"   , ""     , NULL    , T_VOID   , -1, false, _lookupswitch   );
	  def(_fast_binaryswitch   , "fast_binaryswitch"   , ""     , NULL    , T_VOID   , -1, false, _lookupswitch   );
	
	  def(_return_register_finalizer , "return_register_finalizer" , "b"    , NULL    , T_VOID   ,  0, true, _return);
	
	  def(_fast_aldc           , "fast_aldc"           , "bj"   , NULL    , T_OBJECT,   1, true,  _ldc   );
	  def(_fast_aldc_w         , "fast_aldc_w"         , "bJJ"  , NULL    , T_OBJECT,   1, true,  _ldc_w );
	
	  def(_shouldnotreachhere  , "_shouldnotreachhere" , "b"    , NULL    , T_VOID   ,  0, false);
	
  {- -------------------------------------------
  (1) Bytecodes::pd_initialize() を呼んで, 
      プラットフォーム固有の初期化処理を(もしそんなものがあれば)実行しておく
      ---------------------------------------- -}

	  // platform specific JVM bytecodes
	  pd_initialize();
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	  // compare can_trap information for each bytecode with the
	  // can_trap information for the corresponding base bytecode
	  // (if a rewritten bytecode can trap, so must the base bytecode)
	  #ifdef ASSERT
	    { for (int i = 0; i < number_of_codes; i++) {
	        if (is_defined(i)) {
	          Code code = cast(i);
	          Code java = java_code(code);
	          if (can_trap(code) && !can_trap(java))
	            fatal(err_msg("%s can trap => %s can trap, too", name(code),
	                          name(java)));
	        }
	      }
	    }
	  #endif
	
  {- -------------------------------------------
  (1) 初期化が完了したので _is_initialized を true にしておく.
      ---------------------------------------- -}

	  // initialization successful
	  _is_initialized = true;
	}
	
```


