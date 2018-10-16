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

## Resources

Below are links to our CT-Wasm implementations, our evaluation suite, and
mechanizations

#### Reproducing eval results
We provide automated scripts for reproducing the evaluation results in the
`ct-wasm-ports` repository. This repository contains a collection of programs
ported to CT-Wasm.

Simply clone the repository and enter the `eval` directory:

```bash
git clone https://github.com/PLSysSec/ct-wasm-ports
cd ct-wasm-ports/eval
```

**All make commands should be performed within this directory**

#### Validation Performance
We measure the performance of our validator in `ct_node` against a baseline (but
instrumented) node.js. The following command will build `ct_node` and `node` (as necessary), and measure the time to validate a series of CT-Wasm programs.

```
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

#### Statistical Timing (dudect)
We empirically measure the timing characteristics using a modified version of
dudect, which works by collecting samples for a fixed amount of time. By
default, that time is 10 seconds. The following command will build our modified dudect and our extended node.js (`ct_node`) if not already built, and will collect the data into `results/dudect`:

```
DUDE_TIMEOUT=10 make dudect
```

where `DUDE_TIMEOUT` is the sampling time in seconds. The `make` command will
not do anything if you have results already present on disk.

#### Bytecode Sizes
The following command will translate a variety of programs written in CT-Wasm
to bytecode and measure their size:

```
make bytecode_sizes
```

The output can be found in `results/file_sizes.csv`.

#### Node TweetNacl Benchmarks
TweetNacl ships with a series of benchmarks that measure the performance of its various APIs. These benchmarks take a while to execute. We run their benchmarks a number of times, taking the median value, like so:

```
TWEET_TRIALS=10 make tweetnacl
```

This will store the results in `results/node_tweetnacl.csv`.

#### CT-Wasm implementations

- [Reference interpreter](https://github.com/PLSysSec/ct-wasm-spec).
- [Node.js implementation](https://github.com/PLSysSec/ct-wasm-node)
- [Chromium implementation]()

Our implementations fork existing projects, so unless otherwise highlighted,
you should follow the standard build and installation process.

For convenience, we provide [binary
releases](https://github.com/PLSysSec/ct-wasm-spec/releases/artifact) for macOS
and 64 bit Linux. Other platforms should work, but are untested and will need
to build from source.

Chromium is notoriously difficult to maintain a fork for, so we provide a
pre-built binary. It uses the same modifications performed in the V8
subdirectory of our Node.js interpreter.

These releases contain 3 binaries:

 - `ct_node`: a version of node.js that natively supports the use of CT-Wasm.
 This version has been measured by [dude-ct](https://github.com/PLSysSec/dudect) to provide constant-time guarantees.
 - `ct2wasm`: a build of the spec interpreter that supports secrecy stripping through the `-strip` flag.

#### Chromium Builds


### Source Distribution

CT-Wasm efforts are split across a few different repositories:

 - [`ct-wasm-node`](https://github.com/PLSysSec/ct-wasm-node): An implementation in Node/V8
 - [`ct-wasm-ports`](https://github.com/PLSysSec/ct-wasm-ports): Algorithm implementations and evaluation scripts
 - [`tweetnalc-ctwasm`](https://github.com/PLSysSec/tweetnacl-ctwasm): A port of the TweetNacl library with secrecy annotations
 - [`ct-wasm-proofs`](https://github.com/PLSysSec/ct-wasm-proofs): Mechanizations (in Isabelle) of all proofs in the paper

#### Using the source
For building from source, we recommend pulling down
[`ct-wasm-ports`](https://github.com/PLSysSec/ct-wasm-ports) and running
`make tools` from the `eval` directory. This will automatically pull and
build the relevant repositories.

The eval directory also provides a single command to collect all data to
replicate the Evaluation section of the POPL 2019 paper. More details can be
found in the `eval` directory's `README`.

## Summary of spec changes

### New Types
 - `s32`: Secret 32 bit integer
 - `s64`: Secret 64 bit integer

These types come with all integer operations except `div` and `rem` which are
notoriously non-CT and can leak information through partiality.

### New Memory Type
Memories can be either secret or public.

They are declared in text as:
`(memory secret 0 10)`

Secret memories accept and produce secret values but require public indices for stores and loads

```lisp
(module
    (memory secret 1)

    (func $store_example
        (s32.store (i32.const 0) (s32.const 1))))
```

### Declassification
Declassification allows the relabeling of secret data as public. This is inherently unsound but important to operations such as encryption which produce a safely public value out of a secret one.

This is performed by the two operators:
 - `i32.declassify`
 - `i64.declassify`

These operators are only allowed inside **trusted functions**. By default, functions are trusted.
Trust is built into the type of a function like so:

```lisp
(func untrusted (param s32) (result i32)
    ...)
```
