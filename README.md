[//]: # (SPDX-License-Identifier: CC-BY-4.0)
[//]: # (TODO Customize project readme)

# mlkem-rust-libcrux
`mlkem-rust-libcrux` is a portable Rust implementation of ML-KEM which
optionally supports optimizations for AVX2 and NEON platforms.

It originates in the `libcrux` library of formally verified
cryptographic algorithm in Rust.

## Getting Started
To use this implementation of ML-KEM in your Rust project, run

```
cargo add libcrux-ml-kem
```
in your project directory, or add

```
libcrux-ml-kem = "0.0.2"
```
to your `Cargo.toml`.


## Status
The modules
- `mlkem512`
- `mlkem768`
- `mlkem1024`
implement the three parameter sets for ML-KEM defined in FIPS 203. 

Each module provides the following API:
- `generate_key_pair`: to generate an ML-KEM key pair,
- `encapsulate`: to encapsulate a shared secret towards a given ML-KEM public key,
- `decapsulate`: to decapsulate a shared secret from a ciphertext using an ML-KEM private key,
- `validate_public_key`: to perform validation of public keys as required by FIPS 203 prior to encapsulation,
- `validate_private_key`: to perform validation of private keys and ciphertexts as required by FIPS 203 prior to decapsulation.

For detailed documentation, please refer to
[docs.rs](https://docs.rs/libcrux-ml-kem/latest/libcrux_ml_kem/).

### Portable and Optimized Implementations
The crate provides portable, as well as AVX2- and NEON-optimized
implementations of the above API. By defautl, the crate's `build.rs`
will include the portable implementation and one of the optimized
implementations in the build, according to the value of
`CARGO_CFG_TARGET_ARCH`.

In addition, the above functions perform CPU feature detection at
runtime to ensure the most efficient implementation for the given
platform is selected.

It is recommended to rely on the automatic feature detection, but specific
builds can be forced by setting environment variables,
`LIBCRUX_ENABLE_SIMD128=1` or `LIBCRUX_ENABLE_SIMD256=1`.

### Unpacked APIs
The default KEM API described above operates on serialized keys,
i.e. `encapsulate` will take a serialized ML-KEM public key as input
and `decapsulate` will take a serialized ML-KEM private key as input,
and these must be validated before use with `validate_public_key`
and `validate_private_key` for FIPS 203 compliance.

In addition, in each parameter set module, (e.g. `mlkem768`) the crate
provides an API for working with "unpacked" keys, which have already
been deserialized. For some applications it may thus be advantageous
to validate key material once, then deserialized into unpacked
representation once, and to use the the already validated and
deserialized form from then on.

The unpacked APIs are platform dependent, so they can be found in
submodules `mlkem768::portable::unpacked`, `mlkem768::avx2::unpacked`,
`mlkem768::neon::unpacked`, depending on which of these platform
specific modules are part of the build in question.

### Common APIs
The implementation of common APIs across PQCP implementations is
work-in-progress. Progress for this implementation is tracked in issue
[`cryspen/libcrux#894`](https://github.com/cryspen/libcrux/issues/894).

### Crate Features
The crate provides the following features.

Default features:
- `mlkem512`, `mlkem768` & `mlkem1024`: These can be used to select
  individual parameter sets. By default, all parameter sets are
  included.
- `rand`: Whereas the default APIs for `generate_key_pair` and
  `encapsulate` expect external provision of random bytes, this
  feature enables randomized versions of these APIs (in submodules
  `mlkem512::rand`, `mlkem768::rand`, `mlkem1024::rand`) which take an
  `&mut impl rand_core::CryptoRng` argument to generate the required
  randomness internally.
- `default-no-std` & `std`: Disabling default feature `std` provides
  `no_std` support. For convenience `default-no-std` collects all default
  features except `std`.
  
Additional features:
- `kyber`: Provides access to an, as yet, unverified implementation of
  Kyber as submitted in Round 3 of the NIST PQ competition. The Kyber
  APIs follow the general structure of the ML-KEM APIs.
- `check-secret-independence`: All operations on ring elements in the
  portable implementation use the integer types of the
  `libcrux-secrets` crate under the hood. That crate allows checking a
  program operating on these types for secret indepence at compile
  time. Enabling the `check-secret-independence` feature switches on
  this compile-time checking of secret independence. By default, the
  integer types of `libcrux-secrets` transparently fall back on Rust's
  standard integer types.
- `simd128`, `simd256`: These features force a compilation for NEON or
  AVX2 targets, respectively, as discussed above.
- `incremental`: An experimental API, which allows for incremental
  encapsulation.

## Security
As outlined in the description of the `check-secret-independence`
feature above, we leverage the Rust type system to ensure that secret
values are not used in operations that are known to be
non-constant time. While the implementation of constant time
operations is best-effort, as there are no guarantees from the
compiler, we follow established constant-time patterns and validate
constant-time code via inspection of the generated assembly
instructions.

## Verification Status
The code in this crate is formally verified for panic freedom,
correctness and secret independence in F* using the hax toolchain.

TODO

## Performance
We provide a dashboard of benchmark results for the main key
generation, encapsulation and decapsulation APIs across different
platforms and operating systems at
[libcrux.cryspen.com](https://libcrux.cryspen.com).

If you wish to run the benchmarks yourself, you can do so by running
```
cargo bench
```
from the crate root of the `libcrux-ml-kem` crate.

## FAQ
TBD
