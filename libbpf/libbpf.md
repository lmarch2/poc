# Project  
## Description  
A heap buffer overflow vulnerability exists in `bpf_object__init_prog` function of libbpf (`src/libbpf.c` at line 850). This occurs due to missing boundary checks when copying BPF program instructions from a malformed ELF file, leading to memory corruption during `memcpy` operations.  

## libfuzzer report
```
==9==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x51e000119478 at pc 0x5b799afe20f2 bp 0x7fff943e50a0 sp 0x7fff943e4860
READ of size 104 at 0x51e000119478 thread T0
SCARINESS: 26 (multi-byte-read-heap-buffer-overflow)
    #0 0x5b799afe20f1 in __asan_memcpy /src/llvm-project/compiler-rt/lib/asan/asan_interceptors_memintrinsics.cpp:63:3
    #1 0x5b799b072e7b in bpf_object__init_prog /src/libbpf/src/libbpf.c:850:2
    #2 0x5b799b072e7b in bpf_object__add_programs /src/libbpf/src/libbpf.c:922:9
    #3 0x5b799b065140 in bpf_object__elf_collect /src/libbpf/src/libbpf.c:3924:11
    #4 0x5b799b029e1a in bpf_object_open /src/libbpf/src/libbpf.c:8053:16
    #5 0x5b799b02a2aa in bpf_object__open_mem /src/libbpf/src/libbpf.c:8096:20
    #6 0x5b799b0239df in LLVMFuzzerTestOneInput /src/libbpf/fuzz/bpf-object-fuzzer.c:16:8
    #7 0x5b799aed8430 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:614:13
    #8 0x5b799aed7c55 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #9 0x5b799aed9435 in fuzzer::Fuzzer::MutateAndTestOne() /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:760:19
    #10 0x5b799aeda1c5 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:905:5
    #11 0x5b799aec900b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #12 0x5b799aef43e2 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #13 0x728af3596082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 0323ab4806bee6f846d9ad4bccfc29afdca49a58)
    #14 0x5b799aebb88d in _start (/out/bpf-object-fuzzer+0xa688d)

DEDUP_TOKEN: __asan_memcpy--bpf_object__init_prog--bpf_object__add_programs
0x51e000119478 is located 8 bytes before 2624-byte region [0x51e000119480,0x51e000119ec0)
allocated by thread T0 here:
    #0 0x5b799afe41ff in malloc /src/llvm-project/compiler-rt/lib/asan/asan_malloc_linux.cpp:68:3
    #1 0x5b799b121e23 in operator new(unsigned long) cxa_noexception.cpp
    #2 0x5b799aed7c55 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:516:7
    #3 0x5b799aed9435 in fuzzer::Fuzzer::MutateAndTestOne() /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:760:19
    #4 0x5b799aeda1c5 in fuzzer::Fuzzer::Loop(std::__Fuzzer::vector<fuzzer::SizedFile, std::__Fuzzer::allocator<fuzzer::SizedFile>>&) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerLoop.cpp:905:5
    #5 0x5b799aec900b in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerDriver.cpp:914:6
    #6 0x5b799aef43e2 in main /src/llvm-project/compiler-rt/lib/fuzzer/FuzzerMain.cpp:20:10
    #7 0x728af3596082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082) (BuildId: 0323ab4806bee6f846d9ad4bccfc29afdca49a58)

DEDUP_TOKEN: __interceptor_malloc--operator new(unsigned long)--fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool, bool*)
SUMMARY: AddressSanitizer: heap-buffer-overflow /src/libbpf/src/libbpf.c:850:2 in bpf_object__init_prog
Shadow bytes around the buggy address:
  0x51e000119180: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x51e000119200: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x51e000119280: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x51e000119300: fd fd fd fd fd fd fd fa fa fa fa fa fa fa fa fa
  0x51e000119380: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
=>0x51e000119400: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa[fa]
  0x51e000119480: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x51e000119500: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x51e000119580: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x51e000119600: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x51e000119680: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
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
MS: 2 ShuffleBytes-ChangeBit-; base unit: 2ea7f0f9dc53cf0f221e56db3f5b64beae7d862b
artifact_prefix='./'; Test unit written to ./crash-04573b0232eeaed1b2cd9f10e4fadc122c560e7a
```

## reproduce  
1. **Trigger input**: Use the generated malformed BPF file `crash-04573b0232eeaed1b2cd9f10e4fadc122c560e7a`.  
2. **Load with vulnerable libbpf**: Call `bpf_object__open_file` to parse the malicious file.  
3. **Observe crash**: AddressSanitizer reports a heap buffer overflow in `bpf_object__init_prog`.  

## poc
```
// poc.c
#include <stdio.h>
#include <stdlib.h>
#include <bpf/libbpf.h>

int main() {
    const char *crash_file = "crash";

    struct bpf_object *obj = bpf_object__open_file(crash_file, NULL);
    if (!obj) {
        fprintf(stderr, "Failed to open BPF object (expected error)\n");
        return 1;
    }

    int err = bpf_object__load(obj);
    if (err) {
        fprintf(stderr, "Failed to load BPF object (expected error)\n");
        return 1;
    }

    bpf_object__close(obj);
    return 0;
}
```
Specially, run `gcc poc.c -o poc  -I /src/libbpf/include /src/libbpf/src/libbpf.a  -lelf -lz -fsanitize=address` and `./poc`
or run `python3 infra/helper.py reproduce libbpf bpf-object-fuzzer build/out/libbpf/crash-04573b0232eeaed1b2cd9f10e4fadc122c560e7a` via oss-fuzz script

