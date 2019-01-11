# Example Project for GCC RISC-V

This example is for Freedom E300, but can be adapted for other platforms.

## Design

* All library options are defined in `Header/user_settings.h`.
* The memory map is located in the linker file in `linker.ld`.
* Entry point function is `_start` in `crt0.S`.
* The RTC and RNG hardware interfaces need to be implemented for real production applications

## Building

1. Make sure you have `riscv64-unknown-elf` installed.
2. Modify the `Makefile.common`:
  * Use correct toolchain path `TOOLCHAIN`.
  * Use correct architecture 'ARCHFLAGS'.
  * Confirm memory map in linker.ld matches your flash/ram or comment out `SRC_LD = -T./linker.ld` in Makefile.common.
3. Use `make` to build the static library (libwolfssl.a), wolfCrypt test/benchmark and wolfSSL TLS client targets as `.elf` and `.hex` in `/Build`.

## Building for FIPS

1. Request evaluation from wolfSSL by emailing fips@wolfss.com.
2. Modify user_settings.h so section for `HAVE_FIPS` is enabled.
3. Use `make`.
4. Run the wolfCrypt test `./Build/WolfCryptTest.elf` to generate the FIPS boundary HASH

Example:

```
$ Crypt Test
error    test passed!
base64   test passed!
base16   test passed!
asn      test passed!
in my Fips callback, ok = 0, err = -203
message = In Core Integrity check FIPS error
hash = F607C7B983D1D283590448A56381DE460F1E83CB02584F4D77B7F2C583A8F5CD
In core integrity hash check failure, copy above hash
into verifyCore[] in fips_test.c and rebuild
SHA      test failed!
 error = -1802
Crypt Test: Return code -1
```

5. Update the `../../wolfcrypt/src/fips_test.c` array `static const char verifyCore[] = {}` with the correct core hash check.
6. Build again using `make`.
7. Run the wolfCrypt test.

## Performance Tuning Options

These settings are located in `Header/user_settings.h`.

* `DEBUG_WOLFSSL`: Undefine this to disable debug logging.
* `NO_ERROR_STRINGS`: Disables error strings to save code space.
* `NO_INLINE`: Disabling inline function saves about 1KB, but is slower.
* `WOLFSSL_SMALL_STACK`: Enables stack reduction techniques to allocate stack sections over 100 bytes from heap.
* `USE_FAST_MATH`: Uses stack based math, which is faster than the heap based math.
* `ALT_ECC_SIZE`: If using fast math and RSA/DH you can define this to reduce your ECC memory consumption.
* `FP_MAX_BITS`: Is the maximum math size (key size * 2). Used only with `USE_FAST_MATH`.
* `ECC_TIMING_RESISTANT`: Enables timing resistance for ECC and uses slightly less memory.
* `ECC_SHAMIR`: Doubles heap usage, but slightly faster
* `RSA_LOW_MEM`: Half as much memory but twice as slow. Uses Non-CRT method for private key.
* AES GCM: `GCM_SMALL`, `GCM_WORD32` or `GCM_TABLE`: Tunes performance and flash/memory usage.
* `CURVED25519_SMALL`: Enables small versions of Ed/Curve (FE/GE math).
* `USE_SLOW_SHA`: Enables smaller/slower version of SHA.
* `USE_SLOW_SHA256`: About 2k smaller and about 25% slower
* `USE_SLOW_SHA512`: Over twice as small, but 50% slower
* `USE_CERT_BUFFERS_1024` or `USE_CERT_BUFFERS_2048`: Size of RSA certs / keys to test with. 
* `BENCH_EMBEDDED`: Define this if using the wolfCrypt test/benchmark and using a low memory target.
* `ECC_USER_CURVES`: Allows user to defines curve sizes to enable. Default is 256-bit on. To enable others use `HAVE_ECC192`, `HAVE_ECC224`, etc....
* `TFM_ARM`, `TFM_SSE2`, `TFM_AVR32`, `TFM_PPC32`, `TFM_MIPS`, `TFM_X86` or `TFM_X86_64`: These are assembly optimizations available with USE_FAST_MATH.
* Single Precision Math for ARM: See `WOLFSSL_SP`. Optimized math for ARM performance of specific RSA, DH and ECC algorithms.
