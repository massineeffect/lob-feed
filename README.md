# lob-feed

Market data feed handler and limit order book in C++20.

`lob-feed` receives a binary market data stream over UDP, detects and recovers from packet loss and reordering, rebuilds an exact limit order book, computes basic market indicators, and measures its own latency.

> Status: work in progress. Sections marked TBD are filled in only with measured results.

## Why this project

Electronic exchanges publish every order book event as a sequenced binary message. A consumer must apply those events in exact order to keep a correct book, even when the network drops, duplicates or reorders packets. This project implements that pipeline end to end, from a protocol specification to benchmarks.

## Architecture

```
Replayer --UDP (sequenced packets)--> Feed handler --> Order book + indicators
    ^                                      |
    +-------- TCP (retransmit, snapshot) --+

Python reference book --> state hash comparison (differential testing)
```

| Component | Role |
|---|---|
| `replayer` | Replays market data as UDP packets, with configurable loss, duplication, reordering and delay |
| `feed_handler` | Receives packets, tracks sequence numbers, detects gaps, requests recovery |
| `lob_core` | Protocol encoding/decoding, order book, sequencer |
| `tools/ref_book.py` | Deliberately simple reference implementation used to validate the C++ book |

## Protocol

Little-endian, fixed-size messages, prices as `int64` ticks (never floating point). Message types: Add, Cancel, Delete, Replace, Execute. Full specification in [docs/protocol.md](docs/protocol.md).

## Build

Requirements: Linux or WSL2, CMake 3.20+, g++ 13+ or clang 17+.

```bash
cmake --preset debug
cmake --build --preset debug
ctest --preset debug
```

The `debug` preset enables AddressSanitizer and UndefinedBehaviorSanitizer. Use the `release` preset for benchmarks.

## Run

```bash
# TBD: exact commands once the executables exist
./build/release/replayer --help
./build/release/feed_handler --help
```

## Correctness

The C++ order book is compared against the Python reference by hashing the full book state every N messages.

| Scenario | Messages | Divergences |
|---|---|---|
| No loss | TBD | TBD |
| 5% packet loss | TBD | TBD |

## Performance

Measured on: TBD (CPU, OS, compiler, flags).

| Metric | Baseline | Optimized |
|---|---|---|
| Latency p50 | TBD | TBD |
| Latency p99 | TBD | TBD |
| Latency p99.9 | TBD | TBD |
| Throughput (msg/s) | TBD | TBD |

## Design decisions

See [docs/design.md](docs/design.md) for the choices made and the alternatives rejected.

## Limitations

- Single instrument, single thread.
- No matching engine and no order entry: this is a consumer of market data only.
- Kernel bypass (DPDK, AF_XDP) is out of scope; the kernel UDP stack is used.
- Test data is synthetic or a single-symbol sample; results are not representative of a production feed.

## Data

Sample market data is not redistributed in this repository. See [docs/protocol.md](docs/protocol.md) for how to obtain and convert it, subject to the provider's license.

## License

MIT

## Author

Massine Boulanouar - massineboulanouar-droid on GitHub
