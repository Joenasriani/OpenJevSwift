[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift – On-Device Swift AI for Structured Decisions

**Native Swift AI inference for macOS and iOS, with Jev-compatible typed decisions, Core ML and MLX support, and local execution on Apple Silicon.**

[![CI](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml) [![Fixtures](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml) [![Documentation](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml)

OpenJevSwift is a native **Swift on-device AI** library and **Jev-compatible decision server** for
structured probabilistic inference. Ask a model typed questions about text – yes/no, one of several
choices, or a score on a scale – and get model probabilities back instead of generated prose that
must be parsed afterward.

Run the same decision workflow locally behind an HTTP API on an Apple silicon Mac, or embed it
directly in an iPhone or Mac app. OpenJevSwift supports **Core ML** and **MLX** backends, provides a
native alternative to a Python-based OpenJev environment, and keeps decision outputs constrained to
the requested schema.

## Why OpenJevSwift?

- **On-device AI for Swift:** run structured model inference locally in macOS and iOS applications.
- **Typed, schema-constrained decisions:** use `noul` for yes/no probability, `choice` for ranked
  options and `score` for probability across a scale.
- **Core ML and MLX:** serve Verdict and Laya with Core ML, or DiffusionGemma and JevK5 with MLX.
- **Jev API compatibility:** use the Jev-compatible HTTP API and existing TypeSafe SDK workflows.
- **Apple Silicon native:** one `openjev` binary can serve supported models without requiring a
  Python runtime.
- **Measured compatibility:** documented fixtures, benchmark results and model-by-model compatibility
  checks are included in the repository.

Typical uses include local message triage, classification, routing, confidence scoring and other
applications where a Swift app needs structured AI decisions rather than free-form text generation.
The [demo app](#try-the-demo-app) shows these decisions updating as you type.

## Try OpenJevSwift

**See typed AI decisions on an iPhone.** The existing [TriageDemo](Examples/TriageDemo) uses on-device model probabilities to assess message urgency (yes/no), route it to a team (choice) and rate how upset the customer is (score). The demo responds as the message is typed.

![The triage demo answering a billing complaint as it is typed, on an iPhone simulator](docs/assets/triage-demo.gif)

**Get started:** [Run the Mac server](#quick-start) · [Try the iPhone demo](#try-the-demo-app) · [Read the API documentation](https://algorythm-canada.github.io/OpenJevSwift/) · [Check compatibility and limits](docs/compatibility.md).

**Contribute:** See [CONTRIBUTING.md](CONTRIBUTING.md) and [GitHub Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions). Documentation corrections, reproducibility reports and translation reviews are welcome without modifying Swift source code. If OpenJevSwift is useful to you, **star the repository** to improve discovery, **fork it** to test integrations, or add your organization to [ADOPTERS.md](ADOPTERS.md).

OpenJevSwift implements [OpenJev](https://github.com/razorback16/openjev) natively in Swift as an
open, Jev-compatible "System One" AI decision server. Send it a state and typed questions
(`noul`, `choice`, `score`); it reads every answer from a model's probabilities rather than
generating text, so an answer cannot go off-schema: a noul's is the probability of yes, and a
choice's or a score's the probability of every option, with a confidence. One `openjev` binary serves
DiffusionGemma 26B-A4B or JevK5 on MLX, or Verdict or Laya on Core ML, on a Mac, with upstream's
exact wire API, so TypeSafe's SDKs work against it unchanged; the same libraries answer requests inside an
iPhone or Mac app.

OpenJevSwift is an independent project. It is not affiliated with or endorsed by TypeSafe AI (the
makers of Jev), by Google DeepMind or NVIDIA (DiffusionGemma), or by the authors of the other
models it serves. Jev, TypeSafe, Gemma and other names are the property of their respective
owners.

## Quick start

On an Apple silicon Mac with macOS 15 or later and Xcode 27 (the first command is a one-time
install of Xcode's Metal Toolchain, which
[docs/development.md](docs/development.md#xcode-27-needs-the-metal-toolchain) explains):

```bash
xcodebuild -downloadComponent MetalToolchain
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

With Xcode 26.4 to 26.6, add `--build-system swiftbuild` to `swift build`, or the `mlx` and
`jevk5` backends lack MLX's Metal shaders.

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
a launchd job, the logs and the exit statuses.

An app depends on a release and links `OpenJevCore` and the module of each backend it loads:
`OpenJevEncoders` for Verdict and Laya, `OpenJevDiffusionGemma` for DiffusionGemma and
`OpenJevLetterReadout` for JevK5. The DocC article
[Making decisions in an app](https://algorythm-canada.github.io/OpenJevSwift/documentation/openjevcore/gettingstarted/)
loads the same models inside an app:

```swift
dependencies: [
    .package(url: "https://github.com/Algorythm-Canada/OpenJevSwift.git", from: "0.1.0"),
],
targets: [
    .target(
        name: "MyApp",
        dependencies: [
            .product(name: "OpenJevCore", package: "OpenJevSwift"),
            .product(name: "OpenJevEncoders", package: "OpenJevSwift"),
        ]),
]
```

## Try the demo app

[Examples/TriageDemo](Examples/TriageDemo) is an iPhone app that answers upstream's README
example on the device: whether a customer message needs a reply within the hour (`noul`), which
team should handle it (`choice`) and how upset the customer is (`score`), with every option's
probability as a bar, as you type. It needs Xcode 26.4 or later and an iPhone or a simulator with
iOS 18 or later:

1. Close any Xcode window that has the OpenJevSwift package open: Xcode lets only one window use
   a local package.
2. Open `Examples/TriageDemo/TriageDemo.xcodeproj`.
3. To run on an iPhone, choose your team under Signing & Capabilities; the simulator needs none.
4. Run the `TriageDemo` scheme.

The first launch downloads Verdict through the library's own store, about 310 MB from the
openjev-models release and Hugging Face, and checks each file's SHA-256; later launches work
offline. [Examples/TriageDemo/README.md](Examples/TriageDemo/README.md) has the details and the
tests.

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

Release 0.1.0 is the first version a package can depend on; [CHANGELOG.md](CHANGELOG.md) lists
what it ships. Done:

- **Milestones 0 to 5**, every work issue in them closed: the foundations, the decision engine
  core, DiffusionGemma reads on MLX, the Jev-compatible HTTP server with the `openjev` tool, the
  read extensions and images, and text generation with `think` and chat completions (#50 to #53).
- **DiffusionGemma, verified on its checkpoint:** `steps`, `samples` and `sequential` end to end
  (#43, #44 and #45), and image reads, matching upstream's bit for bit on the oracle's kernels
  (#46 to #48).
- **From milestone 6:** Verdict and Laya, and JevK5 (#55), which gives its author's published top
  answer on 230 of JevBench's 231 items (D-052).
- **From milestone 7:** the JevBench comparison with upstream and DiffusionGemma's calibration
  report (#61 and #62).

Not there yet:

- The CLM model, deferred until someone asks for it
  ([D-011](docs/06-decisions.md#d-011-encoder-models-core-ml-for-verdict-and-laya-jevk5-first-among-the-extra-models)).
  If you need it, open an [issue](https://github.com/Algorythm-Canada/OpenJevSwift/issues) or a
  [Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions) post with your use
  case.
- Faster DiffusionGemma reads: #100, #101 and #102.

[docs/08-implementation-plan.md](docs/08-implementation-plan.md) has the milestones and the issue
index.

## Compatibility

Apart from the recorded differences, everything up to the model's probabilities is upstream's byte
for byte: the prompts, the answer templates, the canvases and seeds, the request validation, the
error bodies, the headers and the `/v1/models` listing, all checked against fixtures upstream's own
code writes. The probabilities agree within measured bounds:

- **DiffusionGemma:** within the bounds of decisions D-014 and D-048 on the 63 oracle reads (the
  top label on 91.7% of slots, and on 139 of the 140 where mlx-vlm's top two are at least 0.5
  apart), and between the two servers on 333 JevBench and TypeSafe items.
- **Verdict and Laya:** upstream's top answer on all 666 items.
- **JevK5, on its 8-bit conversion:** its author's published top answer on 230 of JevBench's 231
  items, with the same token counts on all of them.

The differences, among them a stricter JSON parser, the 503 for any backend failure and the
features not built yet, each have a decision record. [docs/compatibility.md](docs/compatibility.md)
has the three tables and the matrix of what runs on macOS, iOS and Linux.

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
- **[CHANGELOG.md](CHANGELOG.md)**: each release and the versioning policy.
- **[SECURITY.md](SECURITY.md)**: how to report a vulnerability, what the code connects to and how
  it handles keys.
- **[ADOPTERS.md](ADOPTERS.md)**: organizations that use OpenJevSwift; add yours with a pull
  request.

## Contributing

Bug reports, compatibility reports, documentation fixes and pull requests are welcome from anyone.
[CONTRIBUTING.md](CONTRIBUTING.md) covers building, testing and opening a pull request. Issues
labelled [help wanted](https://github.com/Algorythm-Canada/OpenJevSwift/labels/help%20wanted) are
open to anyone, and [Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions) is
the place for questions and ideas.

## License and credits

Apache-2.0, the same as upstream OpenJev. Code ported from mlx-vlm (MIT) keeps its copyright notice
in each file's header; [THIRD_PARTY.md](THIRD_PARTY.md) lists every referenced project at its pinned
revision. The models are other people's work and keep their own licenses:
[docs/credits.md](docs/credits.md) credits each, with where this project gets its weights. No
weights are part of this repository.
