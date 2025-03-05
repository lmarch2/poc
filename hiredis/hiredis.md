# Hiredis
## Description  
A heap-buffer-overflow vulnerability exists in the `sdscatlen` function (`sds.c` line 386). This occurs during Redis command formatting when concatenating malformed input data, causing memory access beyond allocated buffer boundaries.

## libfuzzer report  
```
==8==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x502000091997 at pc 0x628c8a590012 bp 0x7ffe614e0470 sp 0x7ffe614dfc30
READ of size 596230 at 0x502000091997 thread T0
SCARINESS: 26 (multi-byte-read-heap-buffer-overflow)
    #0 0x628c8a590011 in __asan_memcpy /src/llvm-project/compiler-rt/lib/asan/asan_interceptors_memintrinsics.cpp:63:3
    #1 0x628c8a5df93a in sdscatlen /src/hiredis/sds.c:386:5
    #2 0x628c8a5d26c8 in redisvFormatCommand /src/hiredis/hiredis.c
    #3 0x628c8a5d467d in redisFormatCommand /src/hiredis/hiredis.c:576:11
    #4 0x628c8a5d18ff in LLVMFuzzerTestOneInput /src/hiredis/fuzzing/format_command_fuzzer.c:51:9
    #5 0x628c8a486350 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:614:13
    #6 0x628c8a485b75 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #7 0x628c8a487355 in fuzzer::Fuzzer::MutateAndTestOne() /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:760:19
    #8 0x628c8a4880e5 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:905:5
    #9 0x628c8a476f2b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #10 0x628c8a4a2302 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #11 0x790b51fc7082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 0323ab4806bee6f846d9ad4bccfc29afdca49a58)
    #12 0x628c8a4697ad in _start (/out/format_command_fuzzer+0x4e7ad)

DEDUP_TOKEN: __asan_memcpy--sdscatlen--redisvFormatCommand
0x502000091997 is located 0 bytes after 7-byte region [0x502000091990,0x502000091997)
allocated by thread T0 here:
    #0 0x628c8a59211f in malloc /src/llvm-project/compiler-rt/lib/asan/asan_malloc_linux.cpp:68:3
    #1 0x628c8a5d18b0 in LLVMFuzzerTestOneInput /src/hiredis/fuzzing/format_command_fuzzer.c:44:15
    #2 0x628c8a486350 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:614:13
    #3 0x628c8a485b75 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #4 0x628c8a487355 in fuzzer::Fuzzer::MutateAndTestOne() /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:760:19
    #5 0x628c8a4880e5 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:905:5
    #6 0x628c8a476f2b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #7 0x628c8a4a2302 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #8 0x790b51fc7082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 0323ab4806bee6f846d9ad4bccfc29afdca49a58)

DEDUP_TOKEN: __interceptor_malloc--LLVMFuzzerTestOneInput--fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long)
SUMMARY: AddressSanitizer: heap-buffer-overflow /src/hiredis/sds.c:386:5 in sdscatlen
Shadow bytes around the buggy address:
  0x502000091700: fa fa fd fa fa fa fd fd fa fa fd fa fa fa fd fa
  0x502000091780: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x502000091800: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x502000091880: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x502000091900: fa fa fd fa fa fa fd fa fa fa fd fa fa fa 06 fa
=>0x502000091980: fa fa[07]fa fa fa fd fa fa fa fd fa fa fa fa fa
  0x502000091a00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x502000091a80: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x502000091b00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x502000091b80: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x502000091c00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==8==ABORTING
MS: 5 EraseBytes-InsertByte-ChangeByte-ChangeBit-ChangeBit-; base unit: 111e36d120834710f3dc7efe89bda9c7fca99c0d
0x35,0x7e,0x25,0x62,0x34,0x25,
5~%b4%
artifact_prefix='./'; Test unit written to ./crash-b84184407f3c266230921208e49070928ddf1392
Base64: NX4lYjQl
```
## reproduce  
run `python3 infra/helper.py reproduce hiredis format_command_fuzzer build/out/hiredis/crash-b84184407f3c266230921208e49070928ddf1392`
or `./format_command_fuzzer crash_input.bin`