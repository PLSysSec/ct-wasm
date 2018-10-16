<img src="./logo.png"/>

------------

#	CT-Wasm: Type-driven Secure Cryptography for the Web Ecosystem

This repository contains all the code and data necessary for building CT-Wasm
and reproducing the results presented in [our paper](https://arxiv.org/abs/1808.01348).

## Abstract

A significant amount of both client and server-side cryptography is implemented
in JavaScript. Despite widespread concerns about its security, no other
language has been able to match the convenience that comes from its ubiquitous
support on the "web ecosystem" - the wide variety of technologies that
collectively underpin the modern World Wide Web. With the new introduction of
the WebAssembly bytecode language (Wasm) into the web ecosystem, we have a
unique opportunity to advance a principled alternative to existing JavaScript
cryptography use cases which does not compromise this convenience.

Constant-Time WebAssembly (CT-Wasm) is a type-driven strict extension
to WebAssembly which facilitates the verifiably secure implementation of
cryptographic algorithms. CT-Wasm's type system ensures that code written in
CT-Wasm is both information flow secure and resistant to timing side channel
attacks; like base Wasm, these guarantees are verifiable in linear time.
Building on an existing Wasm mechanization, we mechanize the full CT-Wasm
specification, prove soundness of the extended type system, implement a
verified type checker, and give several proofs of the language's security
properties. Our security proofs use a novel representation of abstract
information leakage based on quotient types.

We provide two implementations of CT-Wasm: an OCaml reference interpreter and a
native implementation for Node.js and Chromium that extends Google's V8 engine.
We also implement a CT-Wasm to Wasm rewrite tool that allows developers to reap
the benefits of CT-Wasm's type system today, while developing cryptographic
algorithms for base Wasm environments. We evaluate the language, our
implementations, and supporting tools by porting several cryptographic
primitives---Salsa20, SHA-256, and TEA - and the full TweetNaCl library. We
find that CT-Wasm is fast, expressive, and generates code that we
experimentally measure to be constant-time.

## Reproducing Evaluation Results
We provide automated scripts for reproducing the evaluation results in the
[`ct-wasm-ports`](https://github.com/PLSysSec/ct-wasm-ports) repository. This repository contains a collection of programs
ported to CT-Wasm.

First, install prequisites:

### Build Prequisites

- git
- node
- python3 + numpy
- GNU coreutils


#### Transitive Dependencies

(Copied from their respective projects)

**Node.js w/ CT-WASM**

 - gcc and g++ 4.9.4 or newer, or
 - clang and clang++ 3.4.2 or newer (macOS: latest Xcode Command Line Tools)
 - Python 2.6 or 2.7
 - GNU Make 3.81 or newer

**Reference Interpreter**

- Ocaml >= 4.05
- ocamlbuild
- Ocaml num library (for extracted verified compiler). OPAM users can install the num library with: `opam install num`

### Building Evalutation Suite

Simply clone the repository and enter the `eval` directory:

```bash
git clone https://github.com/PLSysSec/ct-wasm-ports
cd ct-wasm-ports/eval
```

*All subsequent make commands below should be performed within this directory*

#### Validation Performance
We measure the performance of our validator as implemented in the Node.js runtime (`ct_node`) against a baseline (but
instrumented) Node.js. The following command will build `ct_node` and `node` (as necessary), and measure the time to validate a series of CT-Wasm programs.

```bash
make validation
```

The output can be found in `results/validation_timing.csv`, measured in milliseconds.
By default, we validation each CT-Wasm program, 10,000 times. If you wish to tweak this number, simply set the `VAL_TRIALS` environment variable like so:

```
VAL_TRIALS=3000 make validation
```

If you've already generated the `csv` you will need to move or delete it for
the `make` command to run.

#### Runtime Performance
We measure performance of our `ct_node` implementation of CT-Wasm against a
baseline `node`. The following command will build both and execute various
algorithms, measuring each 10,000 times:

```
make runtimes
```

Results can be found in `results/crypto_benchmarks.csv` measured in cycles.
Salsa20 and SHA-256 measure the cycles to encrypt 4KB, while TEA measures the
cycles to encrypt 8 bytes.

#### Statistical Timing for Security with dudect
We empirically measure the timing characteristics using a modified version of
[dudect](https://github.com/oreparaz/dudect), which works by collecting samples for a fixed amount of time. By
default, that time is 10 seconds. The following command will build our [modified dudect](https://github.com/PLSysSec/dudect) and our extended Node.js (`ct_node`) if not already built, and will collect the data into `results/dudect`:

```bash
DUDE_TIMEOUT=10 make dudect
```

where `DUDE_TIMEOUT` is the sampling time in seconds. The `make` command will
not do anything if you have results already present on disk.

#### Bytecode Size Overhead
The following command will translate a variety of programs written in CT-Wasm
to bytecode and measure their size:

```
make bytecode_sizes
```

The output can be found in `results/file_sizes.csv`.

#### Node TweetNacl Benchmarks
[TweetNacl](https://github.com/TorstenStueber/TweetNacl-WebAssembly) ships with a series of benchmarks that measure the performance of its various APIs. These benchmarks take a while to execute. We run their benchmarks a number of times (10 bellow), taking the median value, like so:

```
TWEET_TRIALS=10 make tweetnacl
```

This will store the results in `results/node_tweetnacl.csv`.

### Mechanized Proofs

Our mechanization effort and instructions for running Isabelle are described in the  [`ct-wasm-proofs`](https://github.com/PLSysSec/ct-wasm-proofs) repository.

## CT-Wasm Implementations

Though the evaluation scripts above pull the Node.js implementation of CT-Wasm, we include references to all our implementations for completeness:

- [Reference interpreter](https://github.com/PLSysSec/ct-wasm-spec)
- [Node.js implementation](https://github.com/PLSysSec/ct-wasm-node)
- [Chromium implementation](https://github.com/PLSysSec/ct-wasm-chromium)

Our implementations fork existing projects, so unless otherwise highlighted,
you should follow the standard build and installation process.

For convenience, we provide [binary
releases](https://github.com/PLSysSec/ct-wasm-spec/releases/artifact) for macOS
and 64 bit Linux. Other platforms should work, but are untested and will need
to build from source.

These releases contain 3 binaries:

 - `ct_node`: a version of Node.js that natively supports the use of CT-Wasm.
 - `ct2wasm`: tool for removing secrecy labels. Since this is built in the interpreter the `-strip` flag is required.
 - `ct_wasm_spec`: a build of the spec interpreter. The interpreter supports simple secrecy inference via the `-r` flag.


### Source Distribution

CT-Wasm efforts are split across a few different repositories. The evaluation step pulls from these directly. We include the links for complteness.

 - [`ct-wasm-node`](https://github.com/PLSysSec/ct-wasm-node): An implementation of CT-Wasm Node.js/V8.
 - [`ct-wasm-spec`](https://github.com/PLSysSec/ct-wasm-spec): Reference OCaml implementation CT-Wasm with accompanying label stripping and label inference tools.
 - [`ct-wasm-ports`](https://github.com/PLSysSec/ct-wasm-ports): Crypto algorithm implementations and evaluation scripts.
 - [`tweetnacl-ctwasm`](https://github.com/PLSysSec/tweetnacl-ctwasm): A port of the TweetNacl library with secrecy annotations.
 - [`ct-wasm-proofs`](https://github.com/PLSysSec/ct-wasm-proofs): Mechanizations (in Isabelle) of all proofs in the paper.
 - [`dudect`](https://github.com/PLSysSec/dudect): A fork of dudect that's compatible with our instrumented Node.js.
 - [`ct-wasm-chromium`](https://github.com/PLSysSec/ct-wasm-chromium): V8 patches (same as Node.js) for Chromium.
