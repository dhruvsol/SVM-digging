# Invoke Context

Invoke context is part of solana runtime, it provides a context to process transcation when transcations needs to excute on-chain, it keeps check around syscalls, compute limits and sysvars

### Struct in use

- `AllocErr` --> Error struct for memory allocation errors
- `BpfAllocator` --> Berkeley Packet Filter Allocator use for allocating memory to required places, getting use in SysCall context

  - new --> implement new instance of bpfAllocator
  - alloc --> helper function to allocate memory to a Memory Layout

- `SyscallContext` --> Use for tracking System calls, has trace for logs of system call // Unsure
- `SerializedAccountMetadata` --> Seems like part of SyscallContext holds address ( can be memory address for a account ) // Unsure
- `InvokeContext` --> its a stack that holds everything needs to excute a transcation eg: compute for ix's , timing, blockhash, feature set, logs etc.

### Important Functions and what they do

- `prepare_instruction` --> It takes memory stable instruction and process it to different fromat, it also checks for signer previlage, program's validation etc, also checks for deduplication of instrcution accounts
- `process_executable_chain` --> Finds the enterypoint for the program and calls it to excute the instruction method, seems like it creates a Ebpf VM every instruction it process
- `native_invoke` --> Process builtin functions
- `process_instruction` --> handles the compute use when excuting the instruction

## Comparing Changes from agave codebase

- `Enviroment Config` - Invoke context struct has been splitted and some parts are in enviroment config.Gives better readablity but seems helpful to generate specif cases to stress test the runtime
  because it has feature_set in the enviorment config. the mock_invoke context macro has gotten the new addition to init this config

- `process_precompile` - This has been added to support the some kind of internal cache. seems like its part of decuplining of the svm crate its. get_precomiple is use to search. A good performace improvement

## Transcation Context

    This is getting heavily use in the inoke context, seems to be controlling how instructions are getting pass to the invoke context

- `Instruction Context` - Stack of instruction, seems a reverse stake because `get_next_instruction_context` is using `last_mut` so its passing the last index from the stack.

// Theory
Instruction context marks the each instruction with a level to denote inner instruction probably. still figuring out but some kind of parallel invokcation can happen to improve process speed because each instruction is independed. and after doing processing parallel you can join them back and check if no RWLock is not getting overwrite
