---
layout: default
title: HotSpot の内部で使われるデータ構造 ： 詳細 ： opto/ 編
---
[Up](nolpd4szt5.html) [Top](../index.html)

#### HotSpot の内部で使われるデータ構造 ： 詳細 ： opto/ 編

--- 

* [加算に関する高レベル中間語(Ideal)クラス (AddNode, AddINode, AddLNode, AddFNode, AddDNode, AddPNode, OrINode, OrLNode, XorINode, XorLNode, MaxNode, MaxINode, MinINode)](nobAZBxouu.html)
* [AdlcVMDeps クラス ](noAjXkJuhI.html)
* [PhaseCFG, PhaseBlockLayout クラス関連のクラス (Block_Array, Block_List, CFGElement, Block, PhaseCFG, UnionFind, BlockProbPair, CFGLoop, CFGEdge, Trace, PhaseBlockLayout)](noNyJXhcEt.html)
* [C2Compiler クラス ](no-F4zgTod.html)
* [CallGenerator 及びそのサブクラス (CallGenerator, InlineCallGenerator, WarmCallInfo, 及びそれらの補助クラス(ParseGenerator, DirectCallGenerator, DynamicCallGenerator, VirtualCallGenerator, LateInlineCallGenerator, WarmCallGenerator, PredictedCallGenerator, PredictedDynamicCallGenerator, UncommonTrapCallGenerator))](now_Frg7hq.html)
* [メソッド呼び出しに関する高レベル中間語(Ideal)クラス (StartNode, StartOSRNode, ParmNode, ReturnNode, RethrowNode, TailCallNode, TailJumpNode, JVMState, SafePointNode, SafePointScalarObjectNode, CallProjections, CallNode, CallJavaNode, CallStaticJavaNode, CallDynamicJavaNode, CallRuntimeNode, CallLeafNode, CallLeafNoFPNode, AllocateNode, AllocateArrayNode, AbstractLockNode, LockNode, UnlockNode)](nov2u5Q3D5.html)
* [制御構造(Control Flow Graph)に関する高レベル中間語(Ideal)クラス (RegionNode, JProjNode, PhiNode, GotoNode, CProjNode, MultiBranchNode, IfNode, IfTrueNode, IfFalseNode, PCTableNode, JumpNode, JumpProjNode, CatchNode, CatchProjNode, CreateExNode, NeverBranchNode)](nonRisPTfL.html)
* [PhaseChaitin クラス関連のクラス (LRG, LRG_List, PhaseIFG, PhaseChaitin)](nom31wRggL.html)
* [PhaseCoalesce 及びそのサブクラス (PhaseCoalesce, PhaseAggressiveCoalesce, PhaseConservativeCoalesce)](noMgLVYQJ7.html)
* [Compile クラス関連のクラス (Compile, Compile::TracePhase, Compile::AliasType, Compile::Constant, Compile::ConstantTable, 及びそれらの補助クラス(CompileWrapper))](noK9e91eqc.html)
* [定数(CONstant), 条件付き転送(CONditional move), 型変換(CONvert)に関する高レベル中間語(Ideal)クラス (ConNode, ConINode, ConPNode, ConNNode, ConLNode, ConFNode, ConDNode, BinaryNode, CMoveNode, CMoveDNode, CMoveFNode, CMoveINode, CMoveLNode, CMovePNode, CMoveNNode, ConstraintCastNode, CastIINode, CastPPNode, CheckCastPPNode, EncodePNode, DecodeNNode, Conv2BNode, ConvD2FNode, ConvD2INode, ConvD2LNode, ConvF2DNode, ConvF2INode, ConvF2LNode, ConvI2DNode, ConvI2FNode, ConvI2LNode, ConvL2DNode, ConvL2FNode, ConvL2INode, CastX2PNode, CastP2XNode, MemMoveNode, ThreadLocalNode, LoadReturnPCNode, RoundFloatNode, RoundDoubleNode, Opaque1Node, Opaque2Node, PartialSubtypeCheckNode, MoveI2FNode, MoveL2DNode, MoveF2INode, MoveD2LNode, CountBitsNode, CountLeadingZerosINode, CountLeadingZerosLNode, CountTrailingZerosINode, CountTrailingZerosLNode, PopCountINode, PopCountLNode)](nousKk-Za-.html)
* [除算に関する高レベル中間語(Ideal)クラス (DivINode, DivLNode, DivFNode, DivDNode, ModINode, ModLNode, ModFNode, ModDNode, DivModNode, DivModINode, DivModLNode)](nowL8_UL2B.html)
* [PhaseCFG::Dominators() 用の補助クラス (Block_Stack)  ](nouhXcGbgG.html)
* [ConnectionGraph クラス関連のクラス (PointsToNode, ConnectionGraph)](nolyn_5YgA.html)
* [PhaseCFG::GlobalCodeMotion() 用の補助クラス (Node_Backward_Iterator) ](no8Ky93cZO.html)
* [GraphKit クラス関連のクラス (GraphKit, PreserveJVMState, BuildCutout, PreserveReexecuteState)](noyYx1FamR.html)
* [IdealGraphPrinter クラス ](noUGZ50-Nt.html)
* [IdealKit クラス関連のクラス (IdealVariable, IdealKit)](nodRQ82D9E.html)
* [IndexSet クラス関連のクラス (IndexSet, IndexSet::BitBlock, IndexSetIterator)](nouAMDAHD4.html)
* [LibraryIntrinsic クラス関連のクラス (LibraryIntrinsic, LibraryCallKit)](nocZXKuatg.html)
* [PhaseLive クラス ](noRPLlpQjQ.html)
* [同期排他処理に関する高レベル中間語(Ideal)クラス (BoxLockNode, FastLockNode, FastUnlockNode)](noF_P3OUrs.html)
* [PhaseIdealLoop::loop_predication_impl() 用の補助クラス (Invariance) ](nonTUIHjal.html)
* [PhaseIdealLoop クラス, 及びループに関する高レベル中間語(Ideal)クラス (LoopNode, CountedLoopNode, CountedLoopEndNode, LoopLimitNode, IdealLoopTree, PhaseIdealLoop, LoopTreeIterator)](nog6W9J69V.html)
* [低レベル中間語(MachNode)用のクラス (MachOper, MachNode, MachIdealNode, MachTypeNode, MachBreakpointNode, MachConstantBaseNode, MachConstantNode, MachUEPNode, MachPrologNode, MachEpilogNode, MachNopNode, MachSpillCopyNode, MachNullCheckNode, MachProjNode, MachIfNode, MachFastLockNode, MachReturnNode, MachSafePointNode, MachCallNode, MachCallJavaNode, MachCallStaticJavaNode, MachCallDynamicJavaNode, MachCallRuntimeNode, MachCallLeafNode, MachHaltNode, MachTempNode, labelOper, methodOper)](nojDYWLLa8.html)
* [PhaseMacroExpand クラス ](noKfIr9OAQ.html)
* [Matcher クラス関連のクラス (Matcher, 及びその補助クラス(MStack))](nomNJlPgZt.html)
* [メモリアクセスに関する高レベル中間語(Ideal)クラス (MemNode, LoadNode, LoadBNode, LoadUBNode, LoadUSNode, LoadINode, LoadUI2LNode, LoadRangeNode, LoadLNode, LoadL_unalignedNode, LoadFNode, LoadDNode, LoadD_unalignedNode, LoadPNode, LoadNNode, LoadKlassNode, LoadNKlassNode, LoadSNode, StoreNode, StoreBNode, StoreCNode, StoreINode, StoreLNode, StoreFNode, StoreDNode, StorePNode, StoreNNode, StoreCMNode, LoadPLockedNode, LoadLLockedNode, SCMemProjNode, LoadStoreNode, StorePConditionalNode, StoreIConditionalNode, StoreLConditionalNode, CompareAndSwapLNode, CompareAndSwapINode, CompareAndSwapPNode, CompareAndSwapNNode, ClearArrayNode, StrIntrinsicNode, StrCompNode, StrEqualsNode, StrIndexOfNode, AryEqNode, MemBarNode, MemBarAcquireNode, MemBarReleaseNode, MemBarVolatileNode, MemBarCPUOrderNode, InitializeNode, MergeMemNode, MergeMemStream, PrefetchReadNode, PrefetchWriteNode)](no0yb7uLZS.html)
* [乗算に関する高レベル中間語(Ideal)クラス (MulNode, MulINode, MulLNode, MulFNode, MulDNode, MulHiLNode, AndINode, AndLNode, LShiftINode, LShiftLNode, RShiftINode, RShiftLNode, URShiftINode, URShiftLNode)](noIWqMYero.html)
* [MultiNode に関する高レベル中間語(Ideal)クラス (MultiNode, ProjNode)](no9_IdN28o.html)
* [高レベル中間語(Ideal)/低レベル中間語(MachNode)の基底クラス (Node, DUIterator_Common, DUIterator, DUIterator_Fast, DUIterator_Last, SimpleDUIterator, Node_Array, Node_List, Unique_Node_List, Node_Stack, Node_Notes, TypeNode)](no_TtPdTWD.html)
* [OptoReg クラスおよび OptoRegPair クラス (OptoReg, OptoRegPair)](noR2HJuuZ7.html)
* [Compile::Output() 用の補助クラス (Scheduling, NonSafepointEmitter)](no7yPt_kPH.html)
* [Parse 関連のクラス (InlineTree, Parse, Parse::Block, Parse::BytecodeParseHistogram)](no4fDB72o-.html)
* [Parse::do_one_bytecode() 用の補助クラス (SwitchRange) ](noe3ZYHZtN.html)
* [Phase クラス ](no6y_WAgyL.html)
* [PhaseRemoveUseless クラス, PhaseTransform クラスとそのサブクラス, 及びそれらの補助クラス (NodeHash, Type_Array, PhaseRemoveUseless, PhaseTransform, PhaseValues, PhaseGVN, PhaseIterGVN, PhaseCCP, PhasePeephole)](noVFcccfZZ.html)
* [PhaseRegAlloc クラス ](noJpt7ZrMv.html)
* [RegMask クラス ](noYtEw-ppd.html)
* [RootNode に関する高レベル中間語(Ideal)クラス (RootNode, HaltNode)](noLCwOEHvO.html)
* [OptoRuntime クラス, 及び NamedCounter クラス関連のクラス (NamedCounter, BiasedLockingNamedCounter, OptoRuntime)](nofQh7ig_J.html)
* [PhaseStringOpts クラス (PhaseStringOpts, 及びその補助クラス(StringConcat))](nox8KEKmMH.html)
* [減算に関する高レベル中間語(Ideal)クラス (SubNode, SubINode, SubLNode, SubFPNode, SubFNode, SubDNode, CmpNode, CmpINode, CmpUNode, CmpPNode, CmpNNode, CmpLNode, CmpL3Node, CmpFNode, CmpF3Node, CmpDNode, CmpD3Node, BoolNode, AbsNode, AbsINode, AbsFNode, AbsDNode, CmpLTMaskNode, NegNode, NegFNode, NegDNode, CosDNode, SinDNode, TanDNode, AtanDNode, SqrtDNode, ExpDNode, LogDNode, Log10DNode, PowDNode, ReverseBytesINode, ReverseBytesLNode, ReverseBytesUSNode, ReverseBytesSNode)](nokL7da1En.html)
* [SuperWord クラス(とその補助クラス) (DepEdge, DepMem, DepGraph, DepPreds, DepSuccs, SWNodeInfo, SuperWord, SWPointer, OrderedPair)](nos0CPkoTT.html)
* [Type クラス(とそのサブクラス) (Type, TypeF, TypeD, TypeInt, TypeLong, TypeTuple, TypeAry, TypePtr, TypeRawPtr, TypeOopPtr, TypeInstPtr, TypeAryPtr, TypeKlassPtr, TypeNarrowOop, TypeFunc)](noct1eCY52.html)
* [SIMD 命令に関する高レベル中間語(Ideal)クラス (VectorNode, AddVBNode, AddVCNode, AddVSNode, AddVINode, AddVLNode, AddVFNode, AddVDNode, SubVBNode, SubVCNode, SubVSNode, SubVINode, SubVLNode, SubVFNode, SubVDNode, MulVFNode, MulVDNode, DivVFNode, DivVDNode, LShiftVBNode, LShiftVCNode, LShiftVSNode, LShiftVINode, URShiftVBNode, URShiftVCNode, URShiftVSNode, URShiftVINode, AndVNode, OrVNode, XorVNode, VectorLoadNode, Load16BNode, Load8BNode, Load4BNode, Load8CNode, Load4CNode, Load2CNode, Load8SNode, Load4SNode, Load2SNode, Load4INode, Load2INode, Load2LNode, Load4FNode, Load2FNode, Load2DNode, VectorStoreNode, Store16BNode, Store8BNode, Store4BNode, Store8CNode, Store4CNode, Store2CNode, Store4INode, Store2INode, Store2LNode, Store4FNode, Store2FNode, Store2DNode, Replicate16BNode, Replicate8BNode, Replicate4BNode, Replicate8CNode, Replicate4CNode, Replicate2CNode, Replicate8SNode, Replicate4SNode, Replicate2SNode, Replicate4INode, Replicate2INode, Replicate2LNode, Replicate4FNode, Replicate2FNode, Replicate2DNode, PackNode, PackBNode, PackCNode, PackSNode, PackINode, PackLNode, PackFNode, PackDNode, Pack2x1BNode, Pack2x2BNode, ExtractNode, ExtractBNode, ExtractCNode, ExtractSNode, ExtractINode, ExtractLNode, ExtractFNode, ExtractDNode)](noPNylS5jr.html)





