---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.cpp

### 名前(function name)
```
void MacroAssembler::call_VM_helper(Register oop_result, address entry_point, int number_of_arguments, bool check_exceptions) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) コード生成:
      「last_Java_sp の値を計算し, rax レジスタに入れておく.」
  
       (なお, last_Java_pc を last_Java_sp[-1] で取得できるようにするため, 
        MacroAssembler::call_VM_helper() を呼び出す前に
        各 MacroAssembler::call_VM() 内では call 命令でリターンアドレスをスタック上に積んでいる.
  
        そのため, 64 bit モードでは現在の rsp の値に wordSize 足したものを last_Java_sp としている.
        32bit 版では, そのリターンアドレスの後ろに引数が積まれるため、
        引数分も加えたアドレスを last_Java_sp としている.)
      ---------------------------------------- -}

	  // Calculate the value for last_Java_sp
	  // somewhat subtle. call_VM does an intermediate call
	  // which places a return address on the stack just under the
	  // stack pointer as the user finsihed with it. This allows
	  // use to retrieve last_Java_pc from last_Java_sp[-1].
	  // On 32bit we then have to push additional args on the stack to accomplish
	  // the actual requested call. On 64bit call_VM only can use register args
	  // so the only extra space is the return address that call_VM created.
	  // This hopefully explains the calculations here.
	
	#ifdef _LP64
	  // We've pushed one address, correct last_Java_sp
	  lea(rax, Address(rsp, wordSize));
	#else
	  lea(rax, Address(rsp, (1 + number_of_arguments) * wordSize));
	#endif // LP64
	
  {- -------------------------------------------
  (1) コード生成:
      「call_VM_base() が生成するコードにより, 指定のエントリポイントの呼び出しを行う.
       (MacroAssembler::call_VM_base(), またはサブクラスでオーバーライドしたメソッドが呼ばれる)」
      ---------------------------------------- -}

	  call_VM_base(oop_result, noreg, rax, entry_point, number_of_arguments, check_exceptions);
	
	}
	
```


