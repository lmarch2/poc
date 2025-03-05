# fluent-bit
# fluent-bit-uaf
## Description
A use-after-free vulnerability exists in the cfl_list_size function of Fluent Bit (cfl_list.h at line 165).Specifically, cfl_list_size--state_pop_with_cleanup--read_config--flb_cf_yaml_create. This issue occurs due to improper memory management when handling YAML configuration parsing.
## libfuzzer report
```
==9==ERROR: AddressSanitizer: heap-use-after-free on address 0x50b0000018b0 at pc 0x62de6bbb40a5 bp 0x7ffc26638250 sp 0x7ffc26638248
READ of size 8 at 0x50b0000018b0 thread T0
SCARINESS: 51 (8-byte-read-heap-use-after-free)
    #0 0x62de6bbb40a4 in cfl_list_size /src/fluent-bit/lib/cfl/include/cfl/cfl_list.h:165
    #1 0x62de6bbb40a4 in state_pop_with_cleanup /src/fluent-bit/src/config_format/flb_cf_yaml.c:2704:9
    #2 0x62de6bbb40a4 in read_config /src/fluent-bit/src/config_format/flb_cf_yaml.c:2916:25
    #3 0x62de6bbb2d9b in flb_cf_yaml_create /src/fluent-bit/src/config_format/flb_cf_yaml.c:2973:11
    #4 0x62de6bba98ef in LLVMFuzzerTestOneInput /src/fluent-bit/tests/internal/fuzzers/config_yaml_fuzzer.c:56:10
    #5 0x62de6ba60460 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:614:13
    #6 0x62de6ba5fc85 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #7 0x62de6ba61c12 in fuzzer::Fuzzer::ReadAndExecuteSeedCorpora(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:829:7
    #8 0x62de6ba61f02 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:867:3
    #9 0x62de6ba5103b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #10 0x62de6ba7c412 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #11 0x76ee0186a082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 0323ab4806bee6f846d9ad4bccfc29afdca49a58)
    #12 0x62de6ba438bd in _start (/out/flb-it-fuzz-config_yaml_fuzzer_OSSFUZZ+0x708bd)

DEDUP_TOKEN: cfl_list_size--state_pop_with_cleanup--read_config
0x50b0000018b0 is located 96 bytes inside of 104-byte region [0x50b000001850,0x50b0000018b8)
freed by thread T0 here:
    #0 0x62de6bb6bf96 in free /src/llvm-project/compiler-rt/lib/asan/asan_malloc_linux.cpp:52:3
    #1 0x62de6bbb3b1c in flb_free /src/fluent-bit/include/fluent-bit/flb_mem.h:127:5
    #2 0x62de6bbb3b1c in state_destroy /src/fluent-bit/src/config_format/flb_cf_yaml.c:2746:5
    #3 0x62de6bbb3b1c in read_config /src/fluent-bit/src/config_format/flb_cf_yaml.c
    #4 0x62de6bbb81ce in consume_event /src/fluent-bit/src/config_format/flb_cf_yaml.c:934:23
    #5 0x62de6bbb37e0 in read_config /src/fluent-bit/src/config_format/flb_cf_yaml.c:2892:18
    #6 0x62de6bbb2d9b in flb_cf_yaml_create /src/fluent-bit/src/config_format/flb_cf_yaml.c:2973:11
    #7 0x62de6bba98ef in LLVMFuzzerTestOneInput /src/fluent-bit/tests/internal/fuzzers/config_yaml_fuzzer.c:56:10
    #8 0x62de6ba60460 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:614:13
    #9 0x62de6ba5fc85 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #10 0x62de6ba61c12 in fuzzer::Fuzzer::ReadAndExecuteSeedCorpora(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:829:7
    #11 0x62de6ba61f02 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:867:3
    #12 0x62de6ba5103b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #13 0x62de6ba7c412 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #14 0x76ee0186a082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 0323ab4806bee6f846d9ad4bccfc29afdca49a58)

DEDUP_TOKEN: __interceptor_free--flb_free--state_destroy
previously allocated by thread T0 here:
    #0 0x62de6bb6c3f9 in calloc /src/llvm-project/compiler-rt/lib/asan/asan_malloc_linux.cpp:75:3
    #1 0x62de6bbb3190 in flb_calloc /src/fluent-bit/include/fluent-bit/flb_mem.h:95:12
    #2 0x62de6bbb3190 in state_create /src/fluent-bit/src/config_format/flb_cf_yaml.c:2754:13
    #3 0x62de6bbb3190 in state_start /src/fluent-bit/src/config_format/flb_cf_yaml.c:2372:13
    #4 0x62de6bbb3190 in read_config /src/fluent-bit/src/config_format/flb_cf_yaml.c:2836:13
    #5 0x62de6bbb81ce in consume_event /src/fluent-bit/src/config_format/flb_cf_yaml.c:934:23
    #6 0x62de6bbb37e0 in read_config /src/fluent-bit/src/config_format/flb_cf_yaml.c:2892:18
    #7 0x62de6bbb2d9b in flb_cf_yaml_create /src/fluent-bit/src/config_format/flb_cf_yaml.c:2973:11
    #8 0x62de6bba98ef in LLVMFuzzerTestOneInput /src/fluent-bit/tests/internal/fuzzers/config_yaml_fuzzer.c:56:10
    #9 0x62de6ba60460 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:614:13
    #10 0x62de6ba5fc85 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #11 0x62de6ba61c12 in fuzzer::Fuzzer::ReadAndExecuteSeedCorpora(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:829:7
    #12 0x62de6ba61f02 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:867:3
    #13 0x62de6ba5103b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #14 0x62de6ba7c412 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #15 0x76ee0186a082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 0323ab4806bee6f846d9ad4bccfc29afdca49a58)

DEDUP_TOKEN: __interceptor_calloc--flb_calloc--state_create
SUMMARY: AddressSanitizer: heap-use-after-free /src/fluent-bit/lib/cfl/include/cfl/cfl_list.h:165 in cfl_list_size
Shadow bytes around the buggy address:
  0x50b000001600: fa fa fa fa fa fa fa fa fd fd fd fd fd fd fd fd
  0x50b000001680: fd fd fd fd fd fa fa fa fa fa fa fa fa fa fd fd
  0x50b000001700: fd fd fd fd fd fd fd fd fd fd fd fa fa fa fa fa
  0x50b000001780: fa fa fa fa 00 00 00 00 00 00 00 00 00 00 00 00
  0x50b000001800: 00 fa fa fa fa fa fa fa fa fa fd fd fd fd fd fd
=>0x50b000001880: fd fd fd fd fd fd[fd]fa fa fa fa fa fa fa fa fa
  0x50b000001900: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x50b000001980: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x50b000001a00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x50b000001a80: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x50b000001b00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==9==ABORTING
MS: 0 ; base unit: 0000000000000000000000000000000000000000
0x65,0x6e,0x76,0x3a,0xa,0x20,0x20,0x20,0x20,0x6f,0x62,0x73,0x65,0x72,0x76,0x61,0x62,0x69,0x6c,0x69,0x74,0x79,0x3a,0x20,0x63,0x61,0x6c,0x79,0x70,0x74,0x69,0x61,0xa,0xa,0x69,0x6e,0x63,0x6c,0x75,0x64,0x65,0x73,0x3a,0xa,0x20,0x20,0x20,0x20,0x2d,0x20,0x74,0x65,0x73,0x74,0x2f,0x6e,0x65,0x73,0x74,0x65,0x64,0x2e,0x79,0x61,0x6d,0x6c,0xa,
env:\012    observability: calyptia\012\012includes:\012    - test/nested.yaml\012
artifact_prefix='./'; Test unit written to ./crash-03c86884426a5873534f793839cc6607cb7e6fe1
Base64: ZW52OgogICAgb2JzZXJ2YWJpbGl0eTogY2FseXB0aWEKCmluY2x1ZGVzOgogICAgLSB0ZXN0L25lc3RlZC55YW1sCg==
```
## reproduce
run ` bin/fluent-bit -c poc.yaml` to reproduce the crash
```
# poc.yaml
env:
    observability: calyptia
includes:
    - test/nested.yaml
```

![image](https://github.com/user-attachments/assets/8fd21530-50cb-4e1d-93fc-1f12722ebe5f)
