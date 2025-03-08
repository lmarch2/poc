# Description
A heap-buffer-overflow was detected in `compress_chunk_fuzzer `with oss-fuzz on commit `16450518afddcb3139de627157208e49bfef6987 `. It occurred in function `_sw32` that causes an overflow due to an unexpected read from an allocated buffer.
# Details
```
==8==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x504000003030 at pc 0x5ea20ff546bb bp 0x7ffdf88637a0 sp 0x7ffdf8863798
WRITE of size 4 at 0x504000003030 thread T0
SCARINESS: 36 (4-byte-write-heap-buffer-overflow)
    #0 0x5ea20ff546ba in _sw32 /src/c-blosc2/blosc/blosc-private.h:119:23
    #1 0x5ea20ff546ba in serial_blosc /src/c-blosc2/blosc/blosc2.c:1958:7
    #2 0x5ea20ff546ba in do_job /src/c-blosc2/blosc/blosc2.c:2156:15
    #3 0x5ea20ff3e8e9 in blosc_compress_context /src/c-blosc2/blosc/blosc2.c:2480:15
    #4 0x5ea20ff405e7 in blosc2_compress /src/c-blosc2/blosc/blosc2.c:2906:12
    #5 0x5ea20ff34312 in LLVMFuzzerTestOneInput /src/c-blosc2/tests/fuzz/fuzz_compress_chunk.c:44:15
    #6 0x5ea20fde8a90 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:614:13
    #7 0x5ea20fde82b5 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #8 0x5ea20fde9a95 in fuzzer::Fuzzer::MutateAndTestOne() /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:760:19
    #9 0x5ea20fdea825 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:905:5
    #10 0x5ea20fdd966b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #11 0x5ea20fe04a42 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #12 0x799bad113082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 0323ab4806bee6f846d9ad4bccfc29afdca49a58)
    #13 0x5ea20fdcbeed in _start (/out/compress_chunk_fuzzer+0x167eed)

DEDUP_TOKEN: _sw32--serial_blosc--do_job
0x504000003032 is located 0 bytes after 34-byte region [0x504000003010,0x504000003032)
allocated by thread T0 here:
    #0 0x5ea20fef485f in malloc /src/llvm-project/compiler-rt/lib/asan/asan_malloc_linux.cpp:68:3
    #1 0x5ea20ff342cb in LLVMFuzzerTestOneInput /src/c-blosc2/tests/fuzz/fuzz_compress_chunk.c:40:12
    #2 0x5ea20fde8a90 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:614:13
    #3 0x5ea20fde82b5 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #4 0x5ea20fde9a95 in fuzzer::Fuzzer::MutateAndTestOne() /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:760:19
    #5 0x5ea20fdea825 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:905:5
    #6 0x5ea20fdd966b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #7 0x5ea20fe04a42 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #8 0x799bad113082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 0323ab4806bee6f846d9ad4bccfc29afdca49a58)
```
The ASAN report that there is a heap buffer overflow after 34-byte region [0x504000003010,0x504000003032)
In fact, i can reproduce it when debugging with gdb
the function chains `_sw32--serial_blosc--do_job--blosc_compress_context--blosc2_compress`

![Image](https://github.com/user-attachments/assets/b1e2a1f9-5b18-4411-9592-b41770632072)
![Image](https://github.com/user-attachments/assets/1a928ef8-4ade-428f-80dc-07f795910d76)

The overflowed chunk is allocated at `0x504000000190` (Please ignore that the address is slightly different.)
we can see there's a single byte overflow.
 # Reproduce
Run `gdb -args /out/c-blosc2/compress_chunk_fuzzer /out/c-blosc2/crash-ee7a613cbeb45df2ec96c1027db5a43f58b2d9e4`
or use oss-fuzz  `python3 infra/helper.py reproduce c-blosc2 compress_chunk_fuzzer build/out/c-blosc2/crash-ee7a613cbeb45df2ec96c1027db5a43f58b2d9e4`
