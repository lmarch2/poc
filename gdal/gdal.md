# GDAL
## Description  
A use-after-free vulnerability exists in the `OGRSpatialReference::Release` function (`ogrspatialreference.cpp` line 906). This occurs when handling cloned spatial reference objects during vector translation operations, where a prematurely freed `OGRSpatialReference::Private` instance is accessed during layer cleanup.
- Freed by: OGRShapeLayer constructor via operator delete
- Allocated by: OGRSpatialReference::Clone() via operator new

## libfuzzer report  
```
==9==ERROR: AddressSanitizer: heap-use-after-free on address 0x602000093878 at pc 0x000000a61e1e bp 0x7fffebf375a0 sp 0x7fffebf37598
READ of size 8 at 0x602000093878 thread T0
SCARINESS: 51 (8-byte-read-heap-use-after-free)
    #0 0xa61e1d in std::__1::unique_ptr<OGRSpatialReference::Private, std::__1::default_delete<OGRSpatialReference::Private>>::operator->() const /usr/local/bin/../include/c++/v1/memory:2620:19
    #1 0xa6203d in OGRSpatialReference::Dereference() /src/gdal/gdal/ogr/ogrspatialreference.cpp:854:9
    #2 0xa621f9 in OGRSpatialReference::Release() /src/gdal/gdal/ogr/ogrspatialreference.cpp:906:9
    #3 0x14f763c in OGRShapeDataSource::ICreateLayer(char const*, OGRSpatialReference*, OGRwkbGeometryType, char**) /src/gdal/gdal/ogr/ogrsf_frmts/shape/ogrshapedatasource.cpp:811:16
    #4 0x8da351 in SetupTargetLayer::Setup(OGRLayer*, char const*, GDALVectorTranslateOptions*, long long&) /src/gdal/gdal/apps/ogr2ogr_lib.cpp:3729:33
    #5 0x8c6d0e in GDALVectorTranslate /src/gdal/gdal/apps/ogr2ogr_lib.cpp:3111:46
    #6 0x512eb9 in LLVMFuzzerTestOneInput /src/gdal/gdal/./fuzzers/gdal_vector_translate_fuzzer.cpp:119:35
    #7 0x554985 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/libfuzzer/FuzzerLoop.cpp:529:15
    #8 0x5530b9 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool*) /src/libfuzzer/FuzzerLoop.cpp:453:3
    #9 0x5564c3 in fuzzer::Fuzzer::MutateAndTestOne() /src/libfuzzer/FuzzerLoop.cpp:669:19
    #10 0x559888 in fuzzer::Fuzzer::Loop(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, fuzzer::fuzzer_allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>> const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, fuzzer::fuzzer_allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>> const&) /src/libfuzzer/FuzzerLoop.cpp:814:5
    #11 0x52181f in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/libfuzzer/FuzzerDriver.cpp:776:6
    #12 0x5143ec in main /src/libfuzzer/FuzzerMain.cpp:19:10
    #13 0x7661a6bbf082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #14 0x43bc48 in _start (/out/gdal_vector_translate_fuzzer+0x43bc48)

DEDUP_TOKEN: std::__1::unique_ptr<OGRSpatialReference::Private, std::__1::default_delete<OGRSpatialReference::Private>>::operator->() const--OGRSpatialReference::Dereference()--OGRSpatialReference::Release()
0x602000093878 is located 8 bytes inside of 16-byte region [0x602000093870,0x602000093880)
freed by thread T0 here:
    #0 0x50ff38 in operator delete(void*) /src/llvm/projects/compiler-rt/lib/asan/asan_new_delete.cc:166
    #1 0x14fa32e in OGRShapeLayer::OGRShapeLayer(OGRShapeDataSource*, char const*, SHPInfo*, DBFInfo*, OGRSpatialReference*, bool, bool, OGRwkbGeometryType, char**) /src/gdal/gdal/ogr/ogrsf_frmts/shape/ogrshapelayer.cpp:258:18
    #2 0x14f7624 in OGRShapeDataSource::ICreateLayer(char const*, OGRSpatialReference*, OGRwkbGeometryType, char**) /src/gdal/gdal/ogr/ogrsf_frmts/shape/ogrshapedatasource.cpp:807:13
    #3 0x8da351 in SetupTargetLayer::Setup(OGRLayer*, char const*, GDALVectorTranslateOptions*, long long&) /src/gdal/gdal/apps/ogr2ogr_lib.cpp:3729:33
    #4 0x8c6d0e in GDALVectorTranslate /src/gdal/gdal/apps/ogr2ogr_lib.cpp:3111:46
    #5 0x512eb9 in LLVMFuzzerTestOneInput /src/gdal/gdal/./fuzzers/gdal_vector_translate_fuzzer.cpp:119:35
    #6 0x554985 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/libfuzzer/FuzzerLoop.cpp:529:15
    #7 0x5530b9 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool*) /src/libfuzzer/FuzzerLoop.cpp:453:3
    #8 0x5564c3 in fuzzer::Fuzzer::MutateAndTestOne() /src/libfuzzer/FuzzerLoop.cpp:669:19
    #9 0x559888 in fuzzer::Fuzzer::Loop(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, fuzzer::fuzzer_allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>> const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, fuzzer::fuzzer_allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>> const&) /src/libfuzzer/FuzzerLoop.cpp:814:5
    #10 0x52181f in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/libfuzzer/FuzzerDriver.cpp:776:6
    #11 0x5143ec in main /src/libfuzzer/FuzzerMain.cpp:19:10
    #12 0x7661a6bbf082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

DEDUP_TOKEN: operator delete(void*)--OGRShapeLayer::OGRShapeLayer(OGRShapeDataSource*, char const*, SHPInfo*, DBFInfo*, OGRSpatialReference*, bool, bool, OGRwkbGeometryType, char**)--OGRShapeDataSource::ICreateLayer(char const*, OGRSpatialReference*, OGRwkbGeometryType, char**)
previously allocated by thread T0 here:
    #0 0x50f4f8 in operator new(unsigned long) /src/llvm/projects/compiler-rt/lib/asan/asan_new_delete.cc:105
    #1 0xa629b2 in OGRSpatialReference::Clone() const /src/gdal/gdal/ogr/ogrspatialreference.cpp:1162:37
    #2 0x14f74b2 in OGRShapeDataSource::ICreateLayer(char const*, OGRSpatialReference*, OGRwkbGeometryType, char**) /src/gdal/gdal/ogr/ogrsf_frmts/shape/ogrshapedatasource.cpp:780:24
    #3 0x8da351 in SetupTargetLayer::Setup(OGRLayer*, char const*, GDALVectorTranslateOptions*, long long&) /src/gdal/gdal/apps/ogr2ogr_lib.cpp:3729:33
    #4 0x8c6d0e in GDALVectorTranslate /src/gdal/gdal/apps/ogr2ogr_lib.cpp:3111:46
    #5 0x512eb9 in LLVMFuzzerTestOneInput /src/gdal/gdal/./fuzzers/gdal_vector_translate_fuzzer.cpp:119:35
    #6 0x554985 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) /src/libfuzzer/FuzzerLoop.cpp:529:15
    #7 0x5530b9 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long, bool, fuzzer::InputInfo*, bool*) /src/libfuzzer/FuzzerLoop.cpp:453:3
    #8 0x5564c3 in fuzzer::Fuzzer::MutateAndTestOne() /src/libfuzzer/FuzzerLoop.cpp:669:19
    #9 0x559888 in fuzzer::Fuzzer::Loop(std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, fuzzer::fuzzer_allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>> const&, std::__1::vector<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>, fuzzer::fuzzer_allocator<std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>>> const&) /src/libfuzzer/FuzzerLoop.cpp:814:5
    #10 0x52181f in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) /src/libfuzzer/FuzzerDriver.cpp:776:6
    #11 0x5143ec in main /src/libfuzzer/FuzzerMain.cpp:19:10
    #12 0x7661a6bbf082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

DEDUP_TOKEN: operator new(unsigned long)--OGRSpatialReference::Clone() const--OGRShapeDataSource::ICreateLayer(char const*, OGRSpatialReference*, OGRwkbGeometryType, char**)
SUMMARY: AddressSanitizer: heap-use-after-free /usr/local/bin/../include/c++/v1/memory:2620:19 in std::__1::unique_ptr<OGRSpatialReference::Private, std::__1::default_delete<OGRSpatialReference::Private>>::operator->() const
Shadow bytes around the buggy address:
  0x0c048000a6b0: fa fa fa fa fa fa fa fa fa fa 04 fa fa fa fa fa
  0x0c048000a6c0: fa fa fa fa fa fa fd fa fa fa fa fa fa fa fd fd
  0x0c048000a6d0: fa fa fa fa fa fa fd fd fa fa fa fa fa fa fa fa
  0x0c048000a6e0: fa fa fd fd fa fa fa fa fa fa fa fa fa fa fd fa
  0x0c048000a6f0: fa fa fd fd fa fa fa fa fa fa fd fa fa fa fd fa
=>0x0c048000a700: fa fa fa fa fa fa fd fa fa fa fa fa fa fa fd[fd]
  0x0c048000a710: fa fa fa fa fa fa fa fa fa fa fd fa fa fa fa fa
  0x0c048000a720: fa fa fa fa fa fa fd fd fa fa fa fa fa fa fa fa
  0x0c048000a730: fa fa fa fa fa fa fd fd fa fa fd fa fa fa fa fa
  0x0c048000a740: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c048000a750: fa fa fd fd fa fa fa fa fa fa fa fa fa fa fa fa
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
  Shadow gap:              cc
==9==ABORTING
MS: 2 ChangeByte-ChangeBinInt-; base unit: b477889aa0033e4e8de6e4324f88db92e38dfd97
0x46,0x55,0x5a,0x5a,0x45,0x52,0x5f,0x46,0x52,0x49,0x45,0x4e,0x44,0x4c,0x59,0x5f,0x41,0x52,0x43,0x48,0x49,0x56,0x45,0xa,0x2a,0x2a,0x2a,0x4e,0x45,0x57,0x46,0x49,0x4c,0x45,0x2a,0x2a,0x2a,0x3a,0x63,0x6d,0x64,0x2e,0x74,0x78,0x74,0xa,0x6e,0x6f,0x6e,0x5f,0x73,0x73,0x72,0x73,0xf5,0xbe,0xaf,0xac,0x47,0x3a,0x38,0x36,0x35,0x32,0xa,0x2d,0x74,0x5f,0x73,0x72,0x73,0xa,0x4c,0x4f,0x43,0x41,0x4c,0x5f,0x43,0x53,0x5b,0x50,0xff,0xff,0xff,0xff,0xff,0xff,0xff,0xff,0xff,0x53,0x47,0x3a,0x31,0x7e,0x5d,0x49,0x56,0x73,0x65,0x63,0x6f,0x6e,0x64,0xa,0x2a,0x2a,0x2a,0x4e,0x45,0x57,0x46,0x49,0x4c,0x45,0x2a,0x2a,0x2a,0x3a,0x69,0x6e,0x2f,0x66,0x69,0x72,0x73,0x74,0x2e,0x63,0x73,0x76,0xa,0x69,0x66,0x6e,0x5f,0x74,0x69,0x65,0x6c,0x64,0x2c,0x66,0x6c,0x6f,0x61,0x74,0x92,0x92,0x92,0x92,0x92,0x2a,0x2a,0x43,0x4e,0x45,0x57,0x46,0x5f,0x74,0x69,0x65,0x6c,0x64,0x2c,0x66,0x6c,0x6f,0x61,0x74,0x92,0x92,0x92,0x92,0x92,0x92,0x2a,0x2a,0x43,0x4e,0x45,0x57,0x46,0x46,0xff,
FUZZER_FRIENDLY_ARCHIVE\x0a***NEWFILE***:cmd.txt\x0anon_ssrs\xf5\xbe\xaf\xacG:8652\x0a-t_srs\x0aLOCAL_CS[P\xff\xff\xff\xff\xff\xff\xff\xff\xffSG:1~]IVsecond\x0a***NEWFILE***:in/first.csv\x0aifn_tield,float\x92\x92\x92\x92\x92**CNEWF_tield,float\x92\x92\x92\x92\x92\x92**CNEWFF\xff
artifact_prefix='./'; Test unit written to ./crash-d34ee7e1399d2568aface7996c171017b2c0465a
Base64: RlVaWkVSX0ZSSUVORExZX0FSQ0hJVkUKKioqTkVXRklMRSoqKjpjbWQudHh0Cm5vbl9zc3Jz9b6vrEc6ODY1MgotdF9zcnMKTE9DQUxfQ1NbUP///////////1NHOjF+XUlWc2Vjb25kCioqKk5FV0ZJTEUqKio6aW4vZmlyc3QuY3N2Cmlmbl90aWVsZCxmbG9hdJKSkpKSKipDTkVXRl90aWVsZCxmbG9hdJKSkpKSkioqQ05FV0ZG/w==
```
## reproduce 
run `python3 infra/helper.py reproduce gdal gdal_vector_translate_fuzzer build/out/gdal/crash-d34ee7e1399d2568aface7996c171017b2c0465a`