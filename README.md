# OpenJevSwift

[![CI](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml) [![Fixtures](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml) [![Documentation](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml)

## Try OpenJevSwift

**On-device typed AI decisions for Swift apps.** Ask a model a yes/no question, choose between options, or assign a score, and receive structured probabilities instead of free-form generated text.

**See it in action:** [Watch the existing iPhone customer-support triage demo](https://github.com/Algorythm-Canada/OpenJevSwift/blob/main/docs/assets/triage-demo.gif) or [open its example project](https://github.com/Algorythm-Canada/OpenJevSwift/tree/main/Examples/TriageDemo). The demo evaluates urgency, routing and customer sentiment as a message is typed.

**Choose your starting point:** [Run the Mac server](#quick-start) · [Build the iPhone demo](https://github.com/Algorythm-Canada/OpenJevSwift/tree/main/Examples/TriageDemo) · [Read API documentation](https://algorythm-canada.github.io/OpenJevSwift/) · [Review compatibility and limits](https://github.com/Algorythm-Canada/OpenJevSwift/blob/main/docs/compatibility.md).

**Contribute without changing Swift code:** Independently verify the setup instructions, report a reproducible documentation issue, or propose corrections to existing translations through [GitHub Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions) and the [contribution guide](https://github.com/Algorythm-Canada/OpenJevSwift/blob/main/CONTRIBUTING.md). Do not claim adoption or performance results without evidence.

A native Swift implementation of [OpenJev](https://github.com/razorback16/openjev), the open,
Jev-compatible "System One" decision server. Send it a state and typed questions (`noul`,
`choice`, `score`); it reads every answer from a model's probabilities rather than generating
text, so an answer cannot go off-schema: a noul's is the probability of yes, and a choice's or a
score's the probability of every option, with a confidence. One `openjev` binary serves
DiffusionGemma 26B-A4B or JevK5 on MLX, or Verdict or Laya on Core ML, on a Mac, with upstream's
exact wire API, so TypeSafe's SDKs work against it unchanged; the same libraries answer requests inside an
iPhone or Mac app.

OpenJevSwift is an independent project. It is not affiliated with or endorsed by TypeSafe AI (the
makers of Jev), by Google DeepMind or NVIDIA (DiffusionGemma), or by the authors of the other
models it serves. Jev, TypeSafe, Gemma and other names are the property of their respective
owners.

## Quick start

On an Apple silicon Mac with macOS 15 or later, Xcode 27 and its Metal Toolchain component
([docs/development.md](docs/development.md) has the one-time install):

```bash
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

The first start downloads Verdict's converted Core ML package, tokenizer and calibrator (about
310 MB) into Application Support and checks each file's SHA-256; the server listens on
`127.0.0.1:8080` once the model has loaded and warmed up. From another terminal:

```bash
curl -s localhost:8080/v1/systemone -H 'content-type: application/json' \
    -d '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}'
```

```json
{"model":"verdict-1.4","answers":{"urgent":{"type":"noul","noul":0.5910340547561646}},"usage":{"input_tokens":45,"output_tokens":0}}
```

`openjev decide` answers one request without a server and prints the bytes the server sends:

```bash
echo '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}' | .build/release/openjev decide --backend verdict
```

Ctrl-C stops the server gracefully. [docs/deployment.md](docs/deployment.md) covers the settings,
a launchd job, the logs and the exit statuses, and the DocC article "Making decisions in an app"
loads the same models inside an app.

## Requirements

| Backend | Model | Mac | Memory |
|---|---|---|---|
| `verdict` | `verdict-1.4`, 151M parameters, Core ML | Apple silicon, macOS 15 or later | 1.6 GB with the functions one-question reads load, 2.8 GB with all six |
| `laya` | `laya-1.0`, 421M parameters, Core ML | Apple silicon, macOS 15 or later | 4.7 GB with the functions one-question reads load, 8.9 GB with all eight and up to 9.7 GB at peak; with `OPENJEV_ENCODER_FUNCTIONS=2`, 2.1 GB for one-question reads and up to 4.4 GB at peak |
| `mlx` | `openjev-0.1`, DiffusionGemma 26B-A4B, 4-bit, MLX | Apple silicon | about 17 GB to load, the vision tower's 1.06 GiB included (D-054); 17.3 GiB in service with short prompts, measured before the tower loaded, and up to about 3.6 GB more for cached long prompts; 32 GB or more recommended |
| `jevk5` | `jevk5-0.2`, JevK5 (Qwen3.5-4B), 8-bit, MLX | Apple silicon | 6.0 GB once loaded, up to 11.0 GB in service with `OPENJEV_MLX_CACHE_LIMIT_GB=4`; 3.6 and 8.9 GB with the 4-bit conversion |

Building needs Xcode 26.4 or later. The `mlx` backend also needs MLX's Metal shaders, which Swift
Build compiles with the Metal Toolchain: Swift Build is the default with Xcode 27, and Xcode 26
takes `--build-system swiftbuild`. In an app, `OpenJevCore` runs on macOS 14 and
iOS 17 or later, the Verdict and Laya backends on macOS 15 and iOS 18 or later, and JevK5 on
Apple silicon (it builds for iOS; it has not run on an iPhone yet). Linux builds the
core, the server and the `openjev` tool for the tests, without any backend.

## Status

Milestones 0 to 4 are complete, every work issue in them closed: the foundations, the decision
engine core, DiffusionGemma reads on MLX, the Jev-compatible HTTP server with the `openjev` tool,
and the read extensions and images. `steps`, `samples` and `sequential` are verified end to end on
the DiffusionGemma checkpoint (#43, #44 and #45), and images are read on it, matching upstream's
image reads bit for bit on the oracle's kernels (#46 to #48). Verdict and Laya, from milestone 6,
and from milestone 7 the JevBench comparison with upstream and DiffusionGemma's calibration report
(#61 and #62), are done too. JevK5 (#55), also from milestone 6, is served, and gives its author's
published top answer on 230 of JevBench's 231 items (D-052).

Not there yet:

- `think`, a thought before the read: #50, #51 and #52.
- `POST /v1/chat/completions` on DiffusionGemma: the route, its streaming, JSON mode and
  cancellation are in (#53, D-058); it serves once the model generates text, #51.
- The CLM model: #59.
- Release 0.1.0, with a version tag a package can depend on: #65.

[docs/08-implementation-plan.md](docs/08-implementation-plan.md) has the milestones and the issue
index.

## Compatibility

Apart from the recorded differences, everything up to the model's probabilities is upstream's byte
for byte: the prompts, the answer templates, the canvases and seeds, the request validation, the
error bodies, the headers and the `/v1/models` listing, all checked against fixtures upstream's own
code writes. The probabilities agree within measured bounds: DiffusionGemma's within decisions
D-014's and D-048's on the 63 oracle reads (the top label on 91.7% of slots, and on 139 of the 140
where mlx-vlm's top two are at least 0.5 apart), and between the two servers on 333 JevBench and
TypeSafe items; Verdict's and Laya's with upstream's top answer on all 666 items; JevK5's, on its
8-bit conversion, with its author's published top answer on 230 of JevBench's 231 items and the same
token counts on all of them. The differences, among them a stricter JSON parser, the 503 for any
backend failure and the features not built yet, each have a decision record.
[docs/compatibility.md](docs/compatibility.md) has the three tables and the matrix of what runs on
macOS, iOS and Linux.

## Documentation

- **API documentation.** The DocC catalogs of `OpenJevCore`, `OpenJevEncoders`,
  `OpenJevDiffusionGemma`, `OpenJevLetterReadout` and `OpenJevServer`: getting started in an app, the request and answer
  types, implementing a backend, running the server and the configuration reference. The
  Documentation workflow builds them whenever the sources change and publishes them to
  <https://algorythm-canada.github.io/OpenJevSwift/>. `make docs` builds the same site locally
  ([docs/development.md](docs/development.md)).
- **[docs/deployment.md](docs/deployment.md)**: running `openjev serve` on a Mac.
- **[docs/compatibility.md](docs/compatibility.md)**: what is identical to upstream, within
  tolerance, or different.
- **[docs/credits.md](docs/credits.md)**: the models, their authors and licenses.
- **[docs/quality.md](docs/quality.md)** and **[docs/benchmarks.md](docs/benchmarks.md)**: answer
  quality against upstream, and speed and memory.
- **[docs/README.md](docs/README.md)**: the index of the design documents, from how upstream works
  to the decisions and the conformance strategy.

## License and credits

Apache-2.0, the same as upstream OpenJev. Code ported from mlx-vlm (MIT) keeps its copyright notice
in each file's header; [THIRD_PARTY.md](THIRD_PARTY.md) lists every referenced project at its pinned
revision. The models are other people's work and keep their own licenses:
[docs/credits.md](docs/credits.md) credits each, with where this project gets its weights. No
weights are part of this repository.
