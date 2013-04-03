---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： opto/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： opto/

--- 

File Name                                                    | Description
------------------------------------------------------------ | -----------------------------------------------------------------
hotspot/src/share/vm/opto/addnode.cpp                        |  加算に関する高レベル中間語(Ideal)クラスの定義 ([AddNode, AddINode, AddLNode, AddFNode, AddDNode, AddPNode, OrINode, OrLNode, XorINode, XorLNode, MaxNode, MaxINode, MinINode](nobAZBxouu.html))
hotspot/src/share/vm/opto/addnode.hpp                        |  同上
hotspot/src/share/vm/opto/adlcVMDeps.hpp                     |  AdlcVMDeps クラスの定義 ([AdlcVMDeps](noAjXkJuhI.html))
hotspot/src/share/vm/opto/block.cpp                          |  PhaseCFG, PhaseBlockLayout クラス関連のクラスの定義 ([Block_Array, Block_List, CFGElement, Block, PhaseCFG, UnionFind, BlockProbPair, CFGLoop, CFGEdge, Trace, PhaseBlockLayout](noNyJXhcEt.html))
hotspot/src/share/vm/opto/block.hpp                          |  同上
hotspot/src/share/vm/opto/buildOopMap.cpp                    |  Compile::BuildOopMaps() メソッド, 及びその補助メソッドの定義
hotspot/src/share/vm/opto/bytecodeInfo.cpp                   |  InlineTree クラスのメソッド定義 (InlineTree::InlineTree(), InlineTree::should_inline(), InlineTree::should_not_inline(), InlineTree::print_inlining(), InlineTree::ok_to_inline(), InlineTree::build_inline_tree_root(), InlineTree::find_subtree_from_root(), 及びそれらの補助メソッド)
hotspot/src/share/vm/opto/c2_globals.cpp                     |  C2 関連の JVM のコマンドラインオプションの定義 (及びデフォルト値の変更) (※1) (See: [here](no7882y_Y.html) for details)
hotspot/src/share/vm/opto/c2_globals.hpp                     |  同上
hotspot/src/share/vm/opto/c2compiler.cpp                     |  C2Compiler クラスの定義 ([C2Compiler](no-F4zgTod.html))
hotspot/src/share/vm/opto/c2compiler.hpp                     |  同上
hotspot/src/share/vm/opto/callGenerator.cpp                  |  CallGenerator クラス及びそのサブクラスの定義 ([CallGenerator, InlineCallGenerator, WarmCallInfo, 及びそれらの補助クラス(ParseGenerator, DirectCallGenerator, DynamicCallGenerator, VirtualCallGenerator, LateInlineCallGenerator, WarmCallGenerator, PredictedCallGenerator, PredictedDynamicCallGenerator, UncommonTrapCallGenerator)](now_Frg7hq.html))
hotspot/src/share/vm/opto/callGenerator.hpp                  |  同上
hotspot/src/share/vm/opto/callnode.cpp                       |  メソッド呼び出しに関する高レベル中間語(Ideal)クラスの定義 ([StartNode, StartOSRNode, ParmNode, ReturnNode, RethrowNode, TailCallNode, TailJumpNode, JVMState, SafePointNode, SafePointScalarObjectNode, CallProjections, CallNode, CallJavaNode, CallStaticJavaNode, CallDynamicJavaNode, CallRuntimeNode, CallLeafNode, CallLeafNoFPNode, AllocateNode, AllocateArrayNode, AbstractLockNode, LockNode, UnlockNode](nov2u5Q3D5.html))
hotspot/src/share/vm/opto/callnode.hpp                       |  同上
hotspot/src/share/vm/opto/cfgnode.cpp                        |  制御構造(Control Flow Graph)に関する高レベル中間語(Ideal)クラスの定義 ([RegionNode, JProjNode, PhiNode, GotoNode, CProjNode, MultiBranchNode, IfNode, IfTrueNode, IfFalseNode, PCTableNode, JumpNode, JumpProjNode, CatchNode, CatchProjNode, CreateExNode, NeverBranchNode](nonRisPTfL.html))
hotspot/src/share/vm/opto/cfgnode.hpp                        |  同上
hotspot/src/share/vm/opto/chaitin.cpp                        |  PhaseChaitin クラス関連のクラスの定義 ([LRG, LRG_List, PhaseIFG, PhaseChaitin](nom31wRggL.html))
hotspot/src/share/vm/opto/chaitin.hpp                        |  同上
hotspot/src/share/vm/opto/classes.cpp                        |  Ideal クラスの種別一覧の定義 (※2)
hotspot/src/share/vm/opto/classes.hpp                        |  同上
hotspot/src/share/vm/opto/coalesce.cpp                       |  PhaseCoalesce クラス及びそのサブクラスの定義 (※3) ([PhaseCoalesce, PhaseAggressiveCoalesce, PhaseConservativeCoalesce](noMgLVYQJ7.html))
hotspot/src/share/vm/opto/coalesce.hpp                       |  同上
hotspot/src/share/vm/opto/compile.cpp                        |  Compile クラス関連のクラスの定義 ([Compile, Compile::TracePhase, Compile::AliasType, Compile::Constant, Compile::ConstantTable, 及びそれらの補助クラス(CompileWrapper)](noK9e91eqc.html))
hotspot/src/share/vm/opto/compile.hpp                        |  同上
hotspot/src/share/vm/opto/connode.cpp                        |  定数(CONstant), 条件付き転送(CONditional move), 型変換(CONvert)に関する高レベル中間語(Ideal)クラスの定義 ([ConNode, ConINode, ConPNode, ConNNode, ConLNode, ConFNode, ConDNode, BinaryNode, CMoveNode, CMoveDNode, CMoveFNode, CMoveINode, CMoveLNode, CMovePNode, CMoveNNode, ConstraintCastNode, CastIINode, CastPPNode, CheckCastPPNode, EncodePNode, DecodeNNode, Conv2BNode, ConvD2FNode, ConvD2INode, ConvD2LNode, ConvF2DNode, ConvF2INode, ConvF2LNode, ConvI2DNode, ConvI2FNode, ConvI2LNode, ConvL2DNode, ConvL2FNode, ConvL2INode, CastX2PNode, CastP2XNode, MemMoveNode, ThreadLocalNode, LoadReturnPCNode, RoundFloatNode, RoundDoubleNode, Opaque1Node, Opaque2Node, PartialSubtypeCheckNode, MoveI2FNode, MoveL2DNode, MoveF2INode, MoveD2LNode, CountBitsNode, CountLeadingZerosINode, CountLeadingZerosLNode, CountTrailingZerosINode, CountTrailingZerosLNode, PopCountINode, PopCountLNode](nousKk-Za-.html))
hotspot/src/share/vm/opto/connode.hpp                        |  同上
hotspot/src/share/vm/opto/divnode.cpp                        |  除算に関する高レベル中間語(Ideal)クラスの定義 ([DivINode, DivLNode, DivFNode, DivDNode, ModINode, ModLNode, ModFNode, ModDNode, DivModNode, DivModINode, DivModLNode](nowL8_UL2B.html))
hotspot/src/share/vm/opto/divnode.hpp                        |  同上
hotspot/src/share/vm/opto/doCall.cpp                         |  Compile クラス及び Parse クラスの一部のメソッドの定義 (Compile::call_generator(), Parse::do_call(), Parse::catch_inline_exceptions(), Parse::count_compiled_calls(), 及びそれらの補助メソッド)
hotspot/src/share/vm/opto/domgraph.cpp                       |  PhaseCFG クラス及び PhaseIdealLoop クラスの一部のメソッドの定義 (PhaseCFG::Dominators(), PhaseIdealLoop::Dominators(), 及びそれらの補助メソッド／補助クラスの定義) ([Block_Stack](nouhXcGbgG.html))
hotspot/src/share/vm/opto/escape.cpp                         |  ConnectionGraph クラス関連のクラスの定義 ([PointsToNode, ConnectionGraph](nolyn_5YgA.html))
hotspot/src/share/vm/opto/escape.hpp                         |  同上
hotspot/src/share/vm/opto/gcm.cpp                            |  PhaseCFG クラスの一部のメソッドの定義 (PhaseCFG::insert_anti_dependences(), PhaseCFG::latency_from_uses(), PhaseCFG::GlobalCodeMotion(), 及びそれらの補助メソッド／補助クラス) ([Node_Backward_Iterator](no8Ky93cZO.html))
hotspot/src/share/vm/opto/generateOptoStub.cpp               |  GraphKit::gen_stub() の定義
hotspot/src/share/vm/opto/graphKit.cpp                       |  GraphKit クラス関連のクラスの定義 ([GraphKit, PreserveJVMState, BuildCutout, PreserveReexecuteState](noyYx1FamR.html))
hotspot/src/share/vm/opto/graphKit.hpp                       |  同上
hotspot/src/share/vm/opto/idealGraphPrinter.cpp              |  IdealGraphPrinter クラスの定義 ([IdealGraphPrinter](noUGZ50-Nt.html))
hotspot/src/share/vm/opto/idealGraphPrinter.hpp              |  同上
hotspot/src/share/vm/opto/idealKit.cpp                       |  IdealKit クラス関連のクラスの定義 ([IdealVariable, IdealKit](nodRQ82D9E.html))
hotspot/src/share/vm/opto/idealKit.hpp                       |  同上
hotspot/src/share/vm/opto/ifg.cpp                            |  PhaseIFG クラスのメソッド, 及び PhaseChaitin クラスの一部のメソッドの定義
hotspot/src/share/vm/opto/ifnode.cpp                         |  IfNode (一部 IfTrueNode, IfFalseNode) 関連のメソッドの定義
hotspot/src/share/vm/opto/indexSet.cpp                       |  IndexSet クラス関連のクラスの定義 ([IndexSet, IndexSet::BitBlock, IndexSetIterator](nouAMDAHD4.html))
hotspot/src/share/vm/opto/indexSet.hpp                       |  同上
hotspot/src/share/vm/opto/lcm.cpp                            |  Block クラスの一部のメソッドの定義 (Block::implicit_null_check(), Block::schedule_local(), Block::call_catch_cleanup(), 及びそれらの補助メソッド)
hotspot/src/share/vm/opto/library_call.cpp                   |  LibraryIntrinsic 関連のクラス, メソッドの定義 (LibraryIntrinsic, Compile::make_vm_intrinsic(), Compile::register_library_intrinsics(), 及びその補助クラス/補助関数) ([LibraryIntrinsic, LibraryCallKit](nocZXKuatg.html))
hotspot/src/share/vm/opto/live.cpp                           |  PhaseLive クラスの定義 (※4) ([PhaseLive](noRPLlpQjQ.html))
hotspot/src/share/vm/opto/live.hpp                           |  同上
hotspot/src/share/vm/opto/locknode.cpp                       |  同期排他処理に関する高レベル中間語(Ideal)クラスの定義 ([BoxLockNode, FastLockNode, FastUnlockNode](noF_P3OUrs.html))
hotspot/src/share/vm/opto/locknode.hpp                       |  同上
hotspot/src/share/vm/opto/loopPredicate.cpp                  |  PhaseIdealLoop クラス, PhaseIterGVN クラス, 及び IdealLoopTree クラスの一部のメソッドの定義 (PhaseIterGVN::clone_loop_predicates(), PhaseIterGVN::move_loop_predicates(), PhaseIdealLoop::clone_loop_predicates(), PhaseIdealLoop::move_loop_predicates(), PhaseIdealLoop::skip_loop_predicates(), PhaseIdealLoop::find_predicate_insertion_point(), PhaseIdealLoop::find_predicate(), IdealLoopTree::loop_predication(), 及びそれらの補助メソッド／補助クラス) ([Invariance](nonTUIHjal.html))
hotspot/src/share/vm/opto/loopTransform.cpp                  |  IdealLoopTree クラスや PhaseIdealLoop クラスの一部のメソッドの定義 (IdealLoopTree::is_loop_exit(), IdealLoopTree::record_for_igvn(), IdealLoopTree::reassociate_invariants(), IdealLoopTree::policy_range_check(), IdealLoopTre::is_invariant(), PhaseIdealLoop::is_scaled_iv_plus_offset(), IdealLoopTree::iteration_split(), PhaseIdealLoop::do_intrinsify_fill(), 及びそれらの補助メソッド)
hotspot/src/share/vm/opto/loopUnswitch.cpp                   |  Unswitching 処理関連のメソッドの定義  (IdealLoopTree::policy_unswitching(), PhaseIdealLoop::do_unswitching())
hotspot/src/share/vm/opto/loopnode.cpp                       |  PhaseIdealLoop クラスの定義, 及びループに関する高レベル中間語(Ideal)クラスの定義 ([LoopNode, CountedLoopNode, CountedLoopEndNode, LoopLimitNode, IdealLoopTree, PhaseIdealLoop, LoopTreeIterator](nog6W9J69V.html))
hotspot/src/share/vm/opto/loopnode.hpp                       |  同上
hotspot/src/share/vm/opto/loopopts.cpp                       |  PhaseIdealLoop クラスの一部のメソッドの定義 (PhaseIdealLoop::dominated_by(), PhaseIdealLoop::split_if_with_blocks(), PhaseIdealLoop::clone_loop(), PhaseIdealLoop::register_node(), PhaseIdealLoop::short_circuit_if(), PhaseIdealLoop::partial_peel(), PhaseIdealLoop::reorg_offsets(), 及びこれらの補助関数)
hotspot/src/share/vm/opto/machnode.cpp                       |  低レベル中間語(MachNode)用のクラスの定義 ([MachOper, MachNode, MachIdealNode, MachTypeNode, MachBreakpointNode, MachConstantBaseNode, MachConstantNode, MachUEPNode, MachPrologNode, MachEpilogNode, MachNopNode, MachSpillCopyNode, MachNullCheckNode, MachProjNode, MachIfNode, MachFastLockNode, MachReturnNode, MachSafePointNode, MachCallNode, MachCallJavaNode, MachCallStaticJavaNode, MachCallDynamicJavaNode, MachCallRuntimeNode, MachCallLeafNode, MachHaltNode, MachTempNode, labelOper, methodOper](nojDYWLLa8.html))
hotspot/src/share/vm/opto/machnode.hpp                       |  同上
hotspot/src/share/vm/opto/macro.cpp                          |  PhaseMacroExpand クラスの定義 ([PhaseMacroExpand](noKfIr9OAQ.html))
hotspot/src/share/vm/opto/macro.hpp                          |  同上
hotspot/src/share/vm/opto/matcher.cpp                        |  Matcher クラス関連のクラスの定義 ([Matcher, 及びその補助クラス(MStack)](nomNJlPgZt.html))
hotspot/src/share/vm/opto/matcher.hpp                        |  同上
hotspot/src/share/vm/opto/memnode.cpp                        |  メモリアクセスに関する高レベル中間語(Ideal)クラスの定義 ([MemNode, LoadNode, LoadBNode, LoadUBNode, LoadUSNode, LoadINode, LoadUI2LNode, LoadRangeNode, LoadLNode, LoadL_unalignedNode, LoadFNode, LoadDNode, LoadD_unalignedNode, LoadPNode, LoadNNode, LoadKlassNode, LoadNKlassNode, LoadSNode, StoreNode, StoreBNode, StoreCNode, StoreINode, StoreLNode, StoreFNode, StoreDNode, StorePNode, StoreNNode, StoreCMNode, LoadPLockedNode, LoadLLockedNode, SCMemProjNode, LoadStoreNode, StorePConditionalNode, StoreIConditionalNode, StoreLConditionalNode, CompareAndSwapLNode, CompareAndSwapINode, CompareAndSwapPNode, CompareAndSwapNNode, ClearArrayNode, StrIntrinsicNode, StrCompNode, StrEqualsNode, StrIndexOfNode, AryEqNode, MemBarNode, MemBarAcquireNode, MemBarReleaseNode, MemBarVolatileNode, MemBarCPUOrderNode, InitializeNode, MergeMemNode, MergeMemStream, PrefetchReadNode, PrefetchWriteNode](no0yb7uLZS.html))
hotspot/src/share/vm/opto/memnode.hpp                        |  同上
hotspot/src/share/vm/opto/mulnode.cpp                        |  乗算に関する高レベル中間語(Ideal)クラスの定義 ([MulNode, MulINode, MulLNode, MulFNode, MulDNode, MulHiLNode, AndINode, AndLNode, LShiftINode, LShiftLNode, RShiftINode, RShiftLNode, URShiftINode, URShiftLNode](noIWqMYero.html))
hotspot/src/share/vm/opto/mulnode.hpp                        |  同上
hotspot/src/share/vm/opto/multnode.cpp                       |  MultiNode に関する高レベル中間語(Ideal)クラスの定義 ([MultiNode, ProjNode](no9_IdN28o.html))
hotspot/src/share/vm/opto/multnode.hpp                       |  同上
hotspot/src/share/vm/opto/node.cpp                           |  高レベル中間語(Ideal)/低レベル中間語(MachNode)の基底クラスの定義 ([Node, DUIterator_Common, DUIterator, DUIterator_Fast, DUIterator_Last, SimpleDUIterator, Node_Array, Node_List, Unique_Node_List, Node_Stack, Node_Notes, TypeNode](no_TtPdTWD.html))
hotspot/src/share/vm/opto/node.hpp                           |  同上
hotspot/src/share/vm/opto/opcodes.cpp                        |  Ideal クラスの種別一覧の定義 (※2)
hotspot/src/share/vm/opto/opcodes.hpp                        |  同上
hotspot/src/share/vm/opto/optoreg.hpp                        |  OptoReg クラスおよび OptoRegPair クラスの定義 ([OptoReg, OptoRegPair](noR2HJuuZ7.html))
hotspot/src/share/vm/opto/output.cpp                         |  Compile::Output() 及びその補助メソッド／補助クラスの定義 ([Scheduling, NonSafepointEmitter](no7yPt_kPH.html))
hotspot/src/share/vm/opto/output.hpp                         |  同上
hotspot/src/share/vm/opto/parse.hpp                          |  Parse クラス関連のクラスの宣言 ([InlineTree, Parse, Parse::Block, Parse::BytecodeParseHistogram](no4fDB72o-.html))
hotspot/src/share/vm/opto/parse1.cpp                         |  Parse クラス及び Compile クラスの一部のメソッドの定義 (Parse::print_statistics(), Parse::Parse(), Compile::build_start_state(), Compile::return_values(), Compile::rethrow_exceptions(), Parse::throw_to_exit(), Parse::Block::successor_for_bci(), Parse::Block::stack_type_at(), Parse::Block::local_type_at(), Parse::BytecodeParseHistogram::set_initial_state(), Parse::BytecodeParseHistogram::record_change(), Parse::merge(), Parse::merge_new_path(), Parse::merge_exception(), Parse::merge_common(), Parse::return_current(), Parse::add_safepoint(), Parse::dump_bci(), 及びそれらの補助メソッド)
hotspot/src/share/vm/opto/parse2.cpp                         |  Parse::do_one_bytecode() 及びその補助メソッド／補助クラスの定義 (※5) ([SwitchRange](noe3ZYHZtN.html))
hotspot/src/share/vm/opto/parse3.cpp                         |  Parse クラスの一部のメソッドの定義 (Parse::do_field_access(), Parse::push_constant(), Parse::do_anewarray(), Parse::do_newarray(), Parse::do_multianewarray(), 及びそれらの補助メソッド)
hotspot/src/share/vm/opto/parseHelper.cpp                    |  GraphKit クラス及び Parse クラスの一部のメソッドの定義 (GraphKit::make_dtrace_method_entry_exit(), Parse::do_checkcast(), Parse::do_instanceof(), Parse::array_store_check(), Parse::do_new(), Parse::increment_and_test_invocation_counter(), Parse::profile_taken_branch(), Parse::profile_not_taken_branch(), Parse::profile_call(), Parse::profile_ret(), Parse::profile_switch_case(), 及びそれらの補助メソッド) (※6)
hotspot/src/share/vm/opto/phase.cpp                          |  Phase クラスの定義 ([Phase](no6y_WAgyL.html))
hotspot/src/share/vm/opto/phase.hpp                          |  同上
hotspot/src/share/vm/opto/phaseX.cpp                         |  PhaseRemoveUseless クラス, PhaseTransform クラスとそのサブクラス, 及びそれらの補助クラスの定義 ([NodeHash, Type_Array, PhaseRemoveUseless, PhaseTransform, PhaseValues, PhaseGVN, PhaseIterGVN, PhaseCCP, PhasePeephole](noVFcccfZZ.html))
hotspot/src/share/vm/opto/phaseX.hpp                         |  同上
hotspot/src/share/vm/opto/postaloc.cpp                       |  PhaseChaitin::post_allocate_copy_removal() 及びその補助メソッドの定義
hotspot/src/share/vm/opto/reg_split.cpp                      |  PhaseChaitin::Split() 及びその補助メソッドの定義 (※7)
hotspot/src/share/vm/opto/regalloc.cpp                       |  PhaseRegAlloc クラスの定義 ([PhaseRegAlloc](noJpt7ZrMv.html))
hotspot/src/share/vm/opto/regalloc.hpp                       |  同上
hotspot/src/share/vm/opto/regmask.cpp                        |  RegMask クラスの定義 ([RegMask](noYtEw-ppd.html))
hotspot/src/share/vm/opto/regmask.hpp                        |  同上
hotspot/src/share/vm/opto/rootnode.cpp                       |  RootNode に関する高レベル中間語(Ideal)クラスの定義 ([RootNode, HaltNode](noLCwOEHvO.html))
hotspot/src/share/vm/opto/rootnode.hpp                       |  同上
hotspot/src/share/vm/opto/runtime.cpp                        |  OptoRuntime クラス, 及び NamedCounter クラス関連のクラスの定義 ([NamedCounter, BiasedLockingNamedCounter, OptoRuntime](nofQh7ig_J.html))
hotspot/src/share/vm/opto/runtime.hpp                        |  同上
hotspot/src/share/vm/opto/split_if.cpp                       |  PhaseIdealLoop::register_new_node(), PhaseIdealLoop::split_up(), PhaseIdealLoop::do_split_if(), 及びそれらの補助関数の定義
hotspot/src/share/vm/opto/stringopts.cpp                     |  PhaseStringOpts クラスの定義 ([PhaseStringOpts, 及びその補助クラス(StringConcat)](nox8KEKmMH.html))
hotspot/src/share/vm/opto/stringopts.hpp                     |  同上
hotspot/src/share/vm/opto/subnode.cpp                        |  減算に関する高レベル中間語(Ideal)クラスの定義 ([SubNode, SubINode, SubLNode, SubFPNode, SubFNode, SubDNode, CmpNode, CmpINode, CmpUNode, CmpPNode, CmpNNode, CmpLNode, CmpL3Node, CmpFNode, CmpF3Node, CmpDNode, CmpD3Node, BoolNode, AbsNode, AbsINode, AbsFNode, AbsDNode, CmpLTMaskNode, NegNode, NegFNode, NegDNode, CosDNode, SinDNode, TanDNode, AtanDNode, SqrtDNode, ExpDNode, LogDNode, Log10DNode, PowDNode, ReverseBytesINode, ReverseBytesLNode, ReverseBytesUSNode, ReverseBytesSNode](nokL7da1En.html))
hotspot/src/share/vm/opto/subnode.hpp                        |  同上
hotspot/src/share/vm/opto/superword.cpp                      |  SuperWord クラス関連のクラスの定義 ([DepEdge, DepMem, DepGraph, DepPreds, DepSuccs, SWNodeInfo, SuperWord, SWPointer, OrderedPair](nos0CPkoTT.html))
hotspot/src/share/vm/opto/superword.hpp                      |  同上
hotspot/src/share/vm/opto/type.cpp                           |  Type クラス及びそのサブクラスの定義 ([Type, TypeF, TypeD, TypeInt, TypeLong, TypeTuple, TypeAry, TypePtr, TypeRawPtr, TypeOopPtr, TypeInstPtr, TypeAryPtr, TypeKlassPtr, TypeNarrowOop, TypeFunc](noct1eCY52.html))
hotspot/src/share/vm/opto/type.hpp                           |  同上
hotspot/src/share/vm/opto/vectornode.cpp                     |  SIMD 命令に関する高レベル中間語(Ideal)クラスの定義 ([VectorNode, AddVBNode, AddVCNode, AddVSNode, AddVINode, AddVLNode, AddVFNode, AddVDNode, SubVBNode, SubVCNode, SubVSNode, SubVINode, SubVLNode, SubVFNode, SubVDNode, MulVFNode, MulVDNode, DivVFNode, DivVDNode, LShiftVBNode, LShiftVCNode, LShiftVSNode, LShiftVINode, URShiftVBNode, URShiftVCNode, URShiftVSNode, URShiftVINode, AndVNode, OrVNode, XorVNode, VectorLoadNode, Load16BNode, Load8BNode, Load4BNode, Load8CNode, Load4CNode, Load2CNode, Load8SNode, Load4SNode, Load2SNode, Load4INode, Load2INode, Load2LNode, Load4FNode, Load2FNode, Load2DNode, VectorStoreNode, Store16BNode, Store8BNode, Store4BNode, Store8CNode, Store4CNode, Store2CNode, Store4INode, Store2INode, Store2LNode, Store4FNode, Store2FNode, Store2DNode, Replicate16BNode, Replicate8BNode, Replicate4BNode, Replicate8CNode, Replicate4CNode, Replicate2CNode, Replicate8SNode, Replicate4SNode, Replicate2SNode, Replicate4INode, Replicate2INode, Replicate2LNode, Replicate4FNode, Replicate2FNode, Replicate2DNode, PackNode, PackBNode, PackCNode, PackSNode, PackINode, PackLNode, PackFNode, PackDNode, Pack2x1BNode, Pack2x2BNode, ExtractNode, ExtractBNode, ExtractCNode, ExtractSNode, ExtractINode, ExtractLNode, ExtractFNode, ExtractDNode](noPNylS5jr.html))
hotspot/src/share/vm/opto/vectornode.hpp                     |  同上

### 備考(Notes)
* ※1: hpp で補助用のマクロを定義し, cpp で実際にオプションを定義している.

* ※2: opcode.[ch]pp と classes.[ch]pp で Ideal の種別を規程. (cpp ではenum値(Opcode)を返すメソッドである Opcode() の定義, hpp では種別を表す enum の定義)

* ※3: cpp の方は PhaseChaitin の一部のメソッド定義も含む.

* ※4: cpp には PhaseChaitin の verify*() メソッドの定義も2つ含まれている

* ※5: 補助メソッドとしては, do_one_bytecode() から呼ばれる次のようなもの (基本的なバイトコードの処理用のメソッド) もこのファイルに収められている 
       (Parse::do_tableswitch(), Parse::do_lookupswitch(), Parse::modf(), Parse::modd(), Parse::l2f(), Parse::do_irem(), Parse::do_jsr(), Parse::do_ret(), Parse::do_ifnull()))

* ※6: Parse クラスのメソッドは new, instanceof, checkcast の処理と profile取得/invocationcounter操作

* ※7: ただし PhaseChaitin::insert_proj() だけはこのファイル外からも使われている







