# RandomVIREL
RandomVIREL is a proof-of-work (PoW) algorithm that is optimized for general-purpose CPUs. RandomVIREL uses random code execution (hence the name) together with several memory-hard techniques to minimize the efficiency advantage of specialized hardware.

## Overview

RandomVIREL utilizes a virtual machine that executes programs in a special instruction set that consists of integer math, floating point math and branches. These programs can be translated into the CPU's native machine code on the fly (example: [program.asm](doc/program.asm)). At the end, the outputs of the executed programs are consolidated into a 256-bit result using a cryptographic hashing function ([Blake2b](https://blake2.net/)).

RandomVIREL can operate in two main modes with different memory requirements:

* **Fast mode** - requires 2080 MiB of shared memory.
* **Light mode** - requires only 256 MiB of shared memory, but runs significantly slower

Both modes are interchangeable as they give the same results. The fast mode is suitable for "mining", while the light mode is expected to be used only for proof verification.

RandomVIREL is based on [https://github.com/tevador/randomx](RandomX) with reduced scratchpad (1 MiB) and iterations, for faster verification.

## Documentation

Full specification is available in [specs.md](doc/specs.md).

Design description and analysis is available in [design.md](doc/design.md).

## Build

RandomVIREL is written in C++11 and builds a static library with a C API provided by header file [RandomVIREL.h](src/RandomVIREL.h). Minimal API usage example is provided in [api-example1.c](src/tests/api-example1.c). The reference code includes a `RandomVIREL-benchmark` and `RandomVIREL-tests` executables for testing.

### Linux

Build dependencies: `cmake` (minimum 3.5) and `gcc` (minimum version 4.8, but version 7+ is recommended).

To build optimized binaries for your machine, run:
```
git clone https://github.com/tevador/RandomVIREL.git
cd RandomVIREL
mkdir build && cd build
cmake -DARCH=native ..
make
```

To build portable binaries, omit the `ARCH` option when executing cmake.

### Windows

On Windows, it is possible to build using MinGW (same procedure as on Linux) or using Visual Studio (solution file is provided).

### Precompiled binaries

Precompiled `RandomVIREL-benchmark` binaries are available on the [Releases page](https://github.com/tevador/RandomVIREL/releases).

## Proof of work

RandomVIREL was primarily designed as a PoW algorithm for [Virel](https://virel.org/). The recommended usage is following:

* The key `K` is selected to be the hash of a block in the blockchain - this block is called the 'key block'. For optimal mining and verification performance, the key should change every 2048 blocks (~2.8 days) and there should be a delay of 64 blocks (~2 hours) between the key block and the change of the key `K`. This can be achieved by changing the key when `blockHeight % 2048 == 64` and selecting key block such that `keyBlockHeight % 2048 == 0`.
* The input `H` is the standard hashing blob with a selected nonce value.

RandomVIREL was successfully activated on the Monero network on the 30th November 2019.

If you wish to use RandomVIREL as a PoW algorithm for your cryptocurrency, please follow the [configuration guidelines](doc/configuration.md).

**Note**: To achieve ASIC resistance, the key `K` must change and must not be miner-selectable. We recommend to use blockchain data as the key in a similar way to the Monero example above. If blockchain data cannot be used for some reason, use a predefined sequence of keys.


# FAQ

### Which CPU is best for mining RandomVIREL?

Most Intel and AMD CPUs made since 2011 should be fairly efficient at RandomVIREL. More specifically, efficient mining requires:

* 64-bit architecture
* IEEE 754 compliant floating point unit
* Hardware AES support ([AES-NI](https://en.wikipedia.org/wiki/AES_instruction_set) extension for x86, Cryptography extensions for ARMv8)
* 16 KiB of L1 cache, 256 KiB of L2 cache and 2 MiB of L3 cache per mining thread
* Support for large memory pages
* At least 2.5 GiB of free RAM per NUMA node
* Multiple memory channels may be required:
    * DDR3 memory is limited to about 1500-2000 H/s per channel (depending on frequency and timings)
    * DDR4 memory is limited to about 4000-6000 H/s per channel  (depending on frequency and timings)

### Does RandomVIREL facilitate botnets/malware mining or web mining?

Due to the way the algorithm works, mining malware is much easier to detect. [RandomVIREL Sniffer](https://github.com/tevador/RandomVIREL-sniffer) is a proof of concept tool that can detect illicit mining activity on Windows.

Efficient mining requires more than 2 GiB of memory, which also disqualifies many low-end machines such as IoT devices, which are often parts of large botnets.

Web mining is infeasible due to the large memory requirement and the lack of directed rounding support for floating point operations in both Javascript and WebAssembly.

### Since RandomVIREL uses floating point math, does it give reproducible results on different platforms?

RandomVIREL uses only operations that are guaranteed to give correctly rounded results by the [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) standard: addition, subtraction, multiplication, division and square root. Special care is taken to avoid corner cases such as NaN values or denormals.

The reference implementation has been validated on the following platforms:
* x86 (32-bit, little-endian)
* x86-64 (64-bit, little-endian)
* ARMv7+VFPv3 (32-bit, little-endian)
* ARMv8 (64-bit, little-endian)
* PPC64 (64-bit, big-endian)
* RISCV64 (64-bit, little-endian)

### Can FPGAs mine RandomVIREL?

RandomVIREL generates multiple unique programs for every hash, so FPGAs cannot dynamically reconfigure their circuitry because typical FPGA takes tens of seconds to load a bitstream. It is also not possible to generate bitstreams for RandomVIREL programs in advance due to the sheer number of combinations (there are 2<sup>512</sup> unique programs).

Sufficiently large FPGAs can mine RandomVIREL in a [soft microprocessor](https://en.wikipedia.org/wiki/Soft_microprocessor) configuration by emulating a CPU. Under these circumstances, an FPGA will be much less efficient than a CPU or a specialized chip (ASIC).

## Acknowledgements
* [tevador](https://github.com/tevador) - author
* [SChernykh](https://github.com/SChernykh) - contributed significantly to the design of RandomVIREL
* [hyc](https://github.com/hyc) - original idea of using random code execution for PoW
* [Other contributors](https://github.com/tevador/RandomVIREL/graphs/contributors)

RandomVIREL uses some source code from the following 3rd party repositories:
* Argon2d, Blake2b hashing functions: https://github.com/P-H-C/phc-winner-argon2

The author of RandomVIREL declares no competing financial interest.
