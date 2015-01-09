---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/adlc/main.cpp
### 説明(description)

```
//------------------------------main-------------------------------------------
```

### 名前(function name)
```
int main(int argc, char *argv[])
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ArchDesc      AD;             // Architecture Description object
	  globalAD = &AD;
	
	  // ResourceMark  mark;
	  ADLParser    *ADL_Parse;      // ADL Parser object to parse AD file
	
  {- -------------------------------------------
  (1) もしコマンドライン引数がなければ usage() を呼ぶ
      ---------------------------------------- -}

	  // Check for proper arguments
	  if( argc == 1 ) usage(AD);    // No arguments?  Then print usage
	
  {- -------------------------------------------
  (1) コマンドライン引数を処理する
      ---------------------------------------- -}

	  // Read command line arguments and file names
	  for( int i = 1; i < argc; i++ ) { // For all arguments
	    register char *s = argv[i]; // Get option/filename
	
	    if( *s++ == '-' ) {         // It's a flag? (not a filename)
	      if( !*s ) {               // Stand-alone `-' means stdin
	        //********** INSERT CODE HERE **********
	      } else while (*s != '\0') { // While have flags on option
	        switch (*s++) {         // Handle flag
	        case 'd':               // Debug flag
	          AD._dfa_debug += 1;   // Set Debug Flag
	          break;
	        case 'g':               // Debug ad location flag
	          AD._adlocation_debug += 1;       // Set Debug ad location Flag
	          break;
	        case 'o':               // No Output Flag
	          AD._no_output ^= 1;   // Toggle no_output flag
	          break;
	        case 'q':               // Quiet Mode Flag
	          AD._quiet_mode ^= 1;  // Toggle quiet_mode flag
	          break;
	        case 'w':               // Disable Warnings Flag
	          AD._disable_warnings ^= 1; // Toggle disable_warnings flag
	          break;
	        case 'T':               // Option to make DFA as many subroutine calls.
	          AD._dfa_small += 1;   // Set Mode Flag
	          break;
	        case 'c': {             // Set C++ Output file name
	          AD._CPP_file._name = s;
	          const char *base = strip_ext(strdup(s));
	          AD._CPP_CLONE_file._name    = base_plus_suffix(base,"_clone.cpp");
	          AD._CPP_EXPAND_file._name   = base_plus_suffix(base,"_expand.cpp");
	          AD._CPP_FORMAT_file._name   = base_plus_suffix(base,"_format.cpp");
	          AD._CPP_GEN_file._name      = base_plus_suffix(base,"_gen.cpp");
	          AD._CPP_MISC_file._name     = base_plus_suffix(base,"_misc.cpp");
	          AD._CPP_PEEPHOLE_file._name = base_plus_suffix(base,"_peephole.cpp");
	          AD._CPP_PIPELINE_file._name = base_plus_suffix(base,"_pipeline.cpp");
	          s += strlen(s);
	          break;
	        }
	        case 'h':               // Set C++ Output file name
	          AD._HPP_file._name = s; s += strlen(s);
	          break;
	        case 'v':               // Set C++ Output file name
	          AD._VM_file._name = s; s += strlen(s);
	          break;
	        case 'a':               // Set C++ Output file name
	          AD._DFA_file._name = s;
	          AD._bug_file._name = s;
	          s += strlen(s);
	          break;
	        case '#':               // Special internal debug flag
	          AD._adl_debug++;      // Increment internal debug level
	          break;
	        case 's':               // Output which instructions are cisc-spillable
	          AD._cisc_spill_debug = true;
	          break;
	        case 'D':               // Flag Definition
	          {
	            char* flag = s;
	            s += strlen(s);
	            char* def = strchr(flag, '=');
	            if (def == NULL)  def = (char*)"1";
	            else              *def++ = '\0';
	            AD.set_preproc_def(flag, def);
	          }
	          break;
	        case 'U':               // Flag Un-Definition
	          {
	            char* flag = s;
	            s += strlen(s);
	            AD.set_preproc_def(flag, NULL);
	          }
	          break;
	        default:                // Unknown option
	          usage(AD);            // So print usage and exit
	        }                       // End of switch on options...
	      }                         // End of while have options...
	
	    } else {                    // Not an option; must be a filename
	      AD._ADL_file._name = argv[i]; // Set the input filename
	
	      // // Files for storage, based on input file name
	      const char *base = strip_ext(strdup(argv[i]));
	      char       *temp = base_plus_suffix("dfa_",base);
	      AD._DFA_file._name = base_plus_suffix(temp,".cpp");
	      delete temp;
	      temp = base_plus_suffix("ad_",base);
	      AD._CPP_file._name          = base_plus_suffix(temp,".cpp");
	      AD._CPP_CLONE_file._name    = base_plus_suffix(temp,"_clone.cpp");
	      AD._CPP_EXPAND_file._name   = base_plus_suffix(temp,"_expand.cpp");
	      AD._CPP_FORMAT_file._name   = base_plus_suffix(temp,"_format.cpp");
	      AD._CPP_GEN_file._name      = base_plus_suffix(temp,"_gen.cpp");
	      AD._CPP_MISC_file._name     = base_plus_suffix(temp,"_misc.cpp");
	      AD._CPP_PEEPHOLE_file._name = base_plus_suffix(temp,"_peephole.cpp");
	      AD._CPP_PIPELINE_file._name = base_plus_suffix(temp,"_pipeline.cpp");
	      AD._HPP_file._name = base_plus_suffix(temp,".hpp");
	      delete temp;
	      temp = base_plus_suffix("adGlobals_",base);
	      AD._VM_file._name = base_plus_suffix(temp,".hpp");
	      delete temp;
	      temp = base_plus_suffix("bugs_",base);
	      AD._bug_file._name = base_plus_suffix(temp,".out");
	      delete temp;
	    }                           // End of files vs options...
	  }                             // End of while have command line arguments
	
  {- -------------------------------------------
  (1) adlc の入力元ファイルおよび出力先ファイル(AD._*_file)を全て開く.
      失敗したら, adlc をここで終了させる.
      ---------------------------------------- -}

	  // Open files used to store the matcher and its components
	  if (AD.open_files() == 0) return 1; // Open all input/output files
	
  {- -------------------------------------------
  (1) 入力ファイルから内容を読むための FileBuff を作成する
      ---------------------------------------- -}

	  // Build the File Buffer, Parse the input, & Generate Code
	  FileBuff  ADL_Buf(&AD._ADL_file, AD); // Create a file buffer for input file
	
  {- -------------------------------------------
  (1) 入力ファイルの先頭にある Copyright 関係の文章の量を把握しておく 
      (生成したファイルの先頭にも同じ文章をコピーするため)
      ---------------------------------------- -}

	  // Get pointer to legal text at the beginning of AD file.
	  // It will be used in generated ad files.
	  char* legal_text;
	  int legal_sz = get_legal_text(ADL_Buf, &legal_text);
	
  {- -------------------------------------------
  (1) ADLParser を作成し, ADLParser::parse() でパース処理を行う.
      ---------------------------------------- -}

	  ADL_Parse = new ADLParser(ADL_Buf, AD); // Create a parser to parse the buffer
	  ADL_Parse->parse();           // Parse buffer & build description lists
	
  {- -------------------------------------------
  (1) オプションとして d が指定されていた場合は, ArchDesc::dump() で標準エラー出力にダンプ出力を出す.
      ---------------------------------------- -}

	  if( AD._dfa_debug >= 1 ) {    // For higher debug settings, print dump
	    AD.dump();
	  }
	
  {- -------------------------------------------
  (1) 使い終わった ADLParser オブジェクトを破棄
      ---------------------------------------- -}

	  delete ADL_Parse;             // Delete parser
	
  {- -------------------------------------------
  (1) 内容の verify 処理
      ---------------------------------------- -}

	  // Verify that the results of the parse are consistent
	  AD.verify();
	
  {- -------------------------------------------
  (1) ?? #TODO
      ---------------------------------------- -}

	  // Prepare to generate the result files:
	  AD.generateMatchLists();
	  AD.identify_unique_operands();
	  AD.identify_cisc_spill_instructions();
	  AD.identify_short_branches();

  {- -------------------------------------------
  (1) 出力先ファイルの全部の先頭に Copyright 関係の文章をコピーする
      ---------------------------------------- -}

	  // Make sure every file starts with a copyright:
	  AD.addSunCopyright(legal_text, legal_sz, AD._HPP_file._fp);           // .hpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._CPP_file._fp);           // .cpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._CPP_CLONE_file._fp);     // .cpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._CPP_EXPAND_file._fp);    // .cpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._CPP_FORMAT_file._fp);    // .cpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._CPP_GEN_file._fp);       // .cpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._CPP_MISC_file._fp);      // .cpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._CPP_PEEPHOLE_file._fp);  // .cpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._CPP_PIPELINE_file._fp);  // .cpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._VM_file._fp);            // .hpp
	  AD.addSunCopyright(legal_text, legal_sz, AD._DFA_file._fp);           // .cpp

  {- -------------------------------------------
  (1) hpp ファイルについては, 複数回 #include されても問題ないように, 
      "#ifndef ... #define ..." と "#endif" で囲んだ形にする
      (ここで最初の "#ifndef ... #define ..." 部分を出力)
      ---------------------------------------- -}

	  // Add include guards for all .hpp files
	  AD.addIncludeGuardStart(AD._HPP_file, "GENERATED_ADFILES_AD_HPP");        // .hpp
	  AD.addIncludeGuardStart(AD._VM_file, "GENERATED_ADFILES_ADGLOBALS_HPP");  // .hpp

  {- -------------------------------------------
  (1) 各ファイルに対して, 必要な #include 文を出力する
      ---------------------------------------- -}

	  // Add includes
	  AD.addInclude(AD._CPP_file, "precompiled.hpp");
	  AD.addInclude(AD._CPP_file, "adfiles", get_basename(AD._VM_file._name));
	  AD.addInclude(AD._CPP_file, "adfiles", get_basename(AD._HPP_file._name));
	  AD.addInclude(AD._CPP_file, "memory/allocation.inline.hpp");
	  AD.addInclude(AD._CPP_file, "asm/assembler.hpp");
	  AD.addInclude(AD._CPP_file, "code/vmreg.hpp");
	  AD.addInclude(AD._CPP_file, "gc_interface/collectedHeap.inline.hpp");
	  AD.addInclude(AD._CPP_file, "oops/compiledICHolderOop.hpp");
	  AD.addInclude(AD._CPP_file, "oops/markOop.hpp");
	  AD.addInclude(AD._CPP_file, "oops/methodOop.hpp");
	  AD.addInclude(AD._CPP_file, "oops/oop.inline.hpp");
	  AD.addInclude(AD._CPP_file, "oops/oop.inline2.hpp");
	  AD.addInclude(AD._CPP_file, "opto/cfgnode.hpp");
	  AD.addInclude(AD._CPP_file, "opto/locknode.hpp");
	  AD.addInclude(AD._CPP_file, "opto/opcodes.hpp");
	  AD.addInclude(AD._CPP_file, "opto/regalloc.hpp");
	  AD.addInclude(AD._CPP_file, "opto/regmask.hpp");
	  AD.addInclude(AD._CPP_file, "opto/runtime.hpp");
	  AD.addInclude(AD._CPP_file, "runtime/biasedLocking.hpp");
	  AD.addInclude(AD._CPP_file, "runtime/sharedRuntime.hpp");
	  AD.addInclude(AD._CPP_file, "runtime/stubRoutines.hpp");
	  AD.addInclude(AD._CPP_file, "utilities/growableArray.hpp");
	#ifdef TARGET_ARCH_x86
	  AD.addInclude(AD._CPP_file, "assembler_x86.inline.hpp");
	  AD.addInclude(AD._CPP_file, "nativeInst_x86.hpp");
	  AD.addInclude(AD._CPP_file, "vmreg_x86.inline.hpp");
	#endif
	#ifdef TARGET_ARCH_sparc
	  AD.addInclude(AD._CPP_file, "assembler_sparc.inline.hpp");
	  AD.addInclude(AD._CPP_file, "nativeInst_sparc.hpp");
	  AD.addInclude(AD._CPP_file, "vmreg_sparc.inline.hpp");
	#endif
	#ifdef TARGET_ARCH_arm
	  AD.addInclude(AD._CPP_file, "assembler_arm.inline.hpp");
	  AD.addInclude(AD._CPP_file, "nativeInst_arm.hpp");
	  AD.addInclude(AD._CPP_file, "vmreg_arm.inline.hpp");
	#endif
	  AD.addInclude(AD._HPP_file, "memory/allocation.hpp");
	  AD.addInclude(AD._HPP_file, "opto/machnode.hpp");
	  AD.addInclude(AD._HPP_file, "opto/node.hpp");
	  AD.addInclude(AD._HPP_file, "opto/regalloc.hpp");
	  AD.addInclude(AD._HPP_file, "opto/subnode.hpp");
	  AD.addInclude(AD._CPP_CLONE_file, "precompiled.hpp");
	  AD.addInclude(AD._CPP_CLONE_file, "adfiles", get_basename(AD._HPP_file._name));
	  AD.addInclude(AD._CPP_EXPAND_file, "precompiled.hpp");
	  AD.addInclude(AD._CPP_EXPAND_file, "adfiles", get_basename(AD._HPP_file._name));
	  AD.addInclude(AD._CPP_FORMAT_file, "precompiled.hpp");
	  AD.addInclude(AD._CPP_FORMAT_file, "adfiles", get_basename(AD._HPP_file._name));
	  AD.addInclude(AD._CPP_GEN_file, "precompiled.hpp");
	  AD.addInclude(AD._CPP_GEN_file, "adfiles", get_basename(AD._HPP_file._name));
	  AD.addInclude(AD._CPP_GEN_file, "opto/cfgnode.hpp");
	  AD.addInclude(AD._CPP_GEN_file, "opto/locknode.hpp");
	  AD.addInclude(AD._CPP_MISC_file, "precompiled.hpp");
	  AD.addInclude(AD._CPP_MISC_file, "adfiles", get_basename(AD._HPP_file._name));
	  AD.addInclude(AD._CPP_PEEPHOLE_file, "precompiled.hpp");
	  AD.addInclude(AD._CPP_PEEPHOLE_file, "adfiles", get_basename(AD._HPP_file._name));
	  AD.addInclude(AD._CPP_PIPELINE_file, "precompiled.hpp");
	  AD.addInclude(AD._CPP_PIPELINE_file, "adfiles", get_basename(AD._HPP_file._name));
	  AD.addInclude(AD._DFA_file, "precompiled.hpp");
	  AD.addInclude(AD._DFA_file, "adfiles", get_basename(AD._HPP_file._name));
	  AD.addInclude(AD._DFA_file, "opto/matcher.hpp");
	  AD.addInclude(AD._DFA_file, "opto/opcodes.hpp");

  {- -------------------------------------------
  (1) 各ファイルに対して中身を出力する
      ---------------------------------------- -}

	  // Make sure each .cpp file starts with include lines:
	  // files declaring and defining generators for Mach* Objects (hpp,cpp)
	  // Generate the result files:
	  // enumerations, class definitions, object generators, and the DFA
	  // file containing enumeration of machine operands & instructions (hpp)
	  AD.addPreHeaderBlocks(AD._HPP_file._fp);        // .hpp
	  AD.buildMachOperEnum(AD._HPP_file._fp);         // .hpp
	  AD.buildMachOpcodesEnum(AD._HPP_file._fp);      // .hpp
	  AD.buildMachRegisterNumbers(AD._VM_file._fp);   // VM file
	  AD.buildMachRegisterEncodes(AD._HPP_file._fp);  // .hpp file
	  AD.declareRegSizes(AD._HPP_file._fp);           // .hpp
	  AD.build_pipeline_enums(AD._HPP_file._fp);      // .hpp
	  // output definition of class "State"
	  AD.defineStateClass(AD._HPP_file._fp);          // .hpp
	  // file declaring the Mach* classes derived from MachOper and MachNode
	  AD.declareClasses(AD._HPP_file._fp);
	  // declare and define maps: in the .hpp and .cpp files respectively
	  AD.addSourceBlocks(AD._CPP_file._fp);           // .cpp
	  AD.addHeaderBlocks(AD._HPP_file._fp);           // .hpp
	  AD.buildReduceMaps(AD._HPP_file._fp, AD._CPP_file._fp);
	  AD.buildMustCloneMap(AD._HPP_file._fp, AD._CPP_file._fp);
	  // build CISC_spilling oracle and MachNode::cisc_spill() methods
	  AD.build_cisc_spill_instructions(AD._HPP_file._fp, AD._CPP_file._fp);
	  // define methods for machine dependent State, MachOper, and MachNode classes
	  AD.defineClasses(AD._CPP_file._fp);
	  AD.buildMachOperGenerator(AD._CPP_GEN_file._fp);// .cpp
	  AD.buildMachNodeGenerator(AD._CPP_GEN_file._fp);// .cpp
	  // define methods for machine dependent instruction matching
	  AD.buildInstructMatchCheck(AD._CPP_file._fp);  // .cpp
	  // define methods for machine dependent frame management
	  AD.buildFrameMethods(AD._CPP_file._fp);         // .cpp
	
  {- -------------------------------------------
  (1) 各 CPP ファイルの末尾には, 指定のシンボルが
      ビルド時にきちんと #define されているかをチェックするコードを出力
      ---------------------------------------- -}

	  // do this last:
	  AD.addPreprocessorChecks(AD._CPP_file._fp);     // .cpp
	  AD.addPreprocessorChecks(AD._CPP_CLONE_file._fp);     // .cpp
	  AD.addPreprocessorChecks(AD._CPP_EXPAND_file._fp);    // .cpp
	  AD.addPreprocessorChecks(AD._CPP_FORMAT_file._fp);    // .cpp
	  AD.addPreprocessorChecks(AD._CPP_GEN_file._fp);       // .cpp
	  AD.addPreprocessorChecks(AD._CPP_MISC_file._fp);      // .cpp
	  AD.addPreprocessorChecks(AD._CPP_PEEPHOLE_file._fp);  // .cpp
	  AD.addPreprocessorChecks(AD._CPP_PIPELINE_file._fp);  // .cpp
	
  {- -------------------------------------------
  (1) _DFA_file の中身を出力
      ---------------------------------------- -}

	  // define the finite automata that selects lowest cost production
	  AD.buildDFA(AD._DFA_file._fp);

  {- -------------------------------------------
  (1) hpp ファイルについては, 複数回 #include されても問題ないように, 
      "#ifndef ... #define ..." と "#endif" で囲んだ形にする
      (ここで末尾の "#endif" 部分を出力)
      ---------------------------------------- -}

	  // Add include guards for all .hpp files
	  AD.addIncludeGuardEnd(AD._HPP_file, "GENERATED_ADFILES_AD_HPP");        // .hpp
	  AD.addIncludeGuardEnd(AD._VM_file, "GENERATED_ADFILES_ADGLOBALS_HPP");  // .hpp
	
  {- -------------------------------------------
  (1) 入力／出力ファイルを全て閉じる
      ---------------------------------------- -}

	  AD.close_files(0);               // Close all input/output files
	
  {- -------------------------------------------
  (1) 最後にここで統計情報などを出す
      (ということになっているようだが, 現状は何も出していない)
      ---------------------------------------- -}

	  // Final printout and statistics
	  // cout << program;
	
	  if( AD._dfa_debug & 2 ) {    // For higher debug settings, print timing info
	    //    Timer t_stop;
	    //    Timer t_total = t_stop - t_start; // Total running time
	    //    cerr << "\n---Architecture Description Totals---\n";
	    //    cerr << ", Total lines: " << TotalLines;
	    //    float l = TotalLines;
	    //    cerr << "\nTotal Compilation Time: " << t_total << "\n";
	    //    float ft = (float)t_total;
	    //    if( ft > 0.0 ) fprintf(stderr,"Lines/sec: %#5.2f\n", l/ft);
	  }

  {- -------------------------------------------
  (1) サヨナラ, サヨナラ, サヨナラ
      ---------------------------------------- -}

	  return (AD._syntax_errs + AD._semantic_errs + AD._internal_errs); // Bye Bye!!
	}
	
```


