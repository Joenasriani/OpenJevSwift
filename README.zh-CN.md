[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift

[![CI](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml) [![Fixtures](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml) [![Documentation](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml)

> 如果本译文与[英文 README](README.md) 有出入，以英文版本为准。

就一段文本向模型提出带类型的问题（是或否、几个选项中的哪一个，或者在某个量表上的程度），得到的是概率，而不是生成的文本。OpenJevSwift 在本地回答这些问题：在你的 Mac 上通过与 Jev 兼容的 HTTP API 提供服务，或者在你自己的 iPhone 或 Mac 应用内运行。它面向希望在设备端完成这些决策的 Swift 开发者，也面向希望用单个原生二进制文件而不是 Python 环境来使用 OpenJev API 的 Mac 用户。[演示应用](#试用演示应用)展示了它随着你的输入作答。

[OpenJev](https://github.com/razorback16/openjev) 的原生 Swift 实现。OpenJev 是开放的、与 Jev 兼容的“System One”决策服务器。向它发送一个状态和带类型的问题（`noul`、`choice`、`score`）；它从模型的概率中读取每个答案，而不是生成文本，因此答案不会偏离 schema：noul 的答案是“是”的概率，choice 或 score 的答案是每个选项的概率，并附带一个置信度。在 Mac 上，单个 `openjev` 二进制文件就能为 DiffusionGemma 26B-A4B 或 JevK5（基于 MLX）、Verdict 或 Laya（基于 Core ML）提供服务，请求和响应格式与上游完全相同，因此 TypeSafe 的 SDK 无需修改即可与它配合使用；同样的库也能在 iPhone 或 Mac 应用内响应请求。

OpenJevSwift 是一个独立项目。它与 TypeSafe AI（Jev 的开发方）、Google DeepMind 或 NVIDIA（DiffusionGemma）以及它所服务的其他模型的作者均无关联，也未获得他们的认可。Jev、TypeSafe、Gemma 及其他名称均为其各自所有者的财产。

## 快速开始

在搭载 Apple 芯片、运行 macOS 15 或更高版本并装有 Xcode 27 的 Mac 上（第一条命令是一次性安装 Xcode 的 Metal Toolchain，[docs/development.md](docs/development.md#xcode-27-needs-the-metal-toolchain) 对此有说明）：

```bash
xcodebuild -downloadComponent MetalToolchain
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

使用 Xcode 26.4 到 26.6 时，请为 `swift build` 加上 `--build-system swiftbuild`，否则 `mlx` 和 `jevk5` 后端会缺少 MLX 的 Metal 着色器。

首次启动时会把 Verdict 转换后的 Core ML 包、分词器和校准器（约 310 MB）下载到 Application Support，并校验每个文件的 SHA-256；模型加载并预热完成后，服务器开始监听 `127.0.0.1:8080`。在另一个终端中：

```bash
curl -s localhost:8080/v1/systemone -H 'content-type: application/json' \
    -d '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}'
```

```json
{"model":"verdict-1.4","answers":{"urgent":{"type":"noul","noul":0.5910340547561646}},"usage":{"input_tokens":45,"output_tokens":0}}
```

`openjev decide` 无需服务器即可回答单个请求，并输出服务器会发送的字节：

```bash
echo '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}' | .build/release/openjev decide --backend verdict
```

Ctrl-C 会优雅地停止服务器。[docs/deployment.md](docs/deployment.md) 介绍了各项设置、launchd 任务、日志和退出状态。

应用依赖某个发布版本，并链接 `OpenJevCore` 以及它所加载的每个后端的模块：Verdict 和 Laya 对应 `OpenJevEncoders`，DiffusionGemma 对应 `OpenJevDiffusionGemma`，JevK5 对应 `OpenJevLetterReadout`。DocC 文章 [Making decisions in an app](https://algorythm-canada.github.io/OpenJevSwift/documentation/openjevcore/gettingstarted/) 在应用内加载同样的模型：

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

## 试用演示应用

![分诊演示在 iPhone 模拟器上随着输入回答一条账单投诉](docs/assets/triage-demo.gif)

[Examples/TriageDemo](Examples/TriageDemo) 是一个 iPhone 应用，它在设备上回答上游 README 中的示例：一条客户消息是否需要在一小时内回复（`noul`）、应由哪个团队处理（`choice`），以及客户有多不满（`score`），并随着你的输入把每个选项的概率显示为一个条形。它需要 Xcode 26.4 或更高版本，以及运行 iOS 18 或更高版本的 iPhone 或模拟器：

1. 关闭所有打开了 OpenJevSwift 包的 Xcode 窗口：Xcode 只允许一个窗口使用本地包。
2. 打开 `Examples/TriageDemo/TriageDemo.xcodeproj`。
3. 如需在 iPhone 上运行，请在 Signing & Capabilities 下选择你的团队；模拟器则不需要。
4. 运行 `TriageDemo` scheme。

首次启动时会通过库自带的存储下载 Verdict，约 310 MB，来自 openjev-models 发布版本和 Hugging Face，并校验每个文件的 SHA-256；之后的启动可以离线运行。[Examples/TriageDemo/README.md](Examples/TriageDemo/README.md) 中有详细信息和测试。

## 系统要求

| 后端 | 模型 | Mac | 内存 |
|---|---|---|---|
| `verdict` | `verdict-1.4`，151M 参数，Core ML | Apple 芯片，macOS 15 或更高版本 | 加载单问题读取所用的函数时为 1.6 GB，加载全部六个时为 2.8 GB |
| `laya` | `laya-1.0`，421M 参数，Core ML | Apple 芯片，macOS 15 或更高版本 | 加载单问题读取所用的函数时为 4.7 GB，加载全部八个时为 8.9 GB，峰值最高 9.7 GB；设置 `OPENJEV_ENCODER_FUNCTIONS=2` 时，单问题读取为 2.1 GB，峰值最高 4.4 GB |
| `mlx` | `openjev-0.1`，DiffusionGemma 26B-A4B，4-bit，MLX | Apple 芯片 | 加载约需 17 GB，其中包括视觉塔的 1.06 GiB（D-054）；使用短提示词运行服务时为 17.3 GiB（在视觉塔加载前测得），缓存的长提示词最多还需约 3.6 GB；建议 32 GB 或以上 |
| `jevk5` | `jevk5-0.2`，JevK5（Qwen3.5-4B），8-bit，MLX | Apple 芯片 | 加载后为 6.0 GB，设置 `OPENJEV_MLX_CACHE_LIMIT_GB=4` 时运行服务最高 11.0 GB；4-bit 转换版本分别为 3.6 GB 和 8.9 GB |

构建需要 Xcode 26.4 或更高版本。`mlx` 后端还需要 MLX 的 Metal 着色器，由 Swift Build 使用 Metal Toolchain 编译：Xcode 27 默认使用 Swift Build，Xcode 26 则需加上 `--build-system swiftbuild`。在应用中，`OpenJevCore` 可在 macOS 14 和 iOS 17 或更高版本上运行，Verdict 和 Laya 后端可在 macOS 15 和 iOS 18 或更高版本上运行，JevK5 可在 Apple 芯片上运行（它可以为 iOS 构建，但尚未在 iPhone 上运行过）。在 Linux 上会构建核心、服务器和 `openjev` 工具以供测试，不包含任何后端。

## 项目状态

0.1.0 是第一个可供包依赖的发布版本；[CHANGELOG.md](CHANGELOG.md) 列出了它包含的内容。已完成：

- **里程碑 0 到 5**，其中的每个工作 issue 都已关闭：基础部分、决策引擎核心、MLX 上的 DiffusionGemma 读取、带 `openjev` 工具的 Jev 兼容 HTTP 服务器、读取扩展和图像，以及支持 `think` 和 chat completions 的文本生成（#50 到 #53）。
- **DiffusionGemma，已在其检查点上验证：** `steps`、`samples` 和 `sequential` 端到端（#43、#44 和 #45），以及在 oracle 的内核上与上游逐位一致的图像读取（#46 到 #48）。
- **来自里程碑 6：** Verdict 和 Laya，以及 JevK5（#55），它在 JevBench 的 231 个条目中有 230 个给出其作者公布的首选答案（D-052）。
- **来自里程碑 7：** 与上游的 JevBench 对比，以及 DiffusionGemma 的校准报告（#61 和 #62）。

尚未完成：

- CLM 模型，推迟到有人提出需要时再做（[D-011](docs/06-decisions.md#d-011-encoder-models-core-ml-for-verdict-and-laya-jevk5-first-among-the-extra-models)）。如果你需要它，请提交一个 [issue](https://github.com/Algorythm-Canada/OpenJevSwift/issues) 或在 [Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions) 中发帖，说明你的使用场景。
- 更快的 DiffusionGemma 读取：#100、#101 和 #102。

[docs/08-implementation-plan.md](docs/08-implementation-plan.md) 中有各个里程碑和 issue 索引。

## 兼容性

除已记录的差异外，直到模型概率为止的一切都与上游逐字节一致：提示词、答案模板、画布和种子、请求验证、错误响应体、标头以及 `/v1/models` 列表，全部都对照上游自身代码写出的 fixture 进行了检查。概率在实测范围内一致：

- **DiffusionGemma：** 在 63 次 oracle 读取上处于决策 D-014 和 D-048 的范围内（91.7% 的槽位首选标签一致；在 mlx-vlm 前两名相差至少 0.5 的 140 个槽位中，有 139 个一致），两个服务器之间在 333 个 JevBench 和 TypeSafe 条目上也是如此。
- **Verdict 和 Laya：** 在全部 666 个条目上都给出上游的首选答案。
- **JevK5（8-bit 转换版本）：** 在 JevBench 的 231 个条目中有 230 个给出其作者公布的首选答案，且所有条目的 token 数都相同。

这些差异（其中包括更严格的 JSON 解析器、任何后端故障都返回 503，以及尚未实现的功能）各有一条决策记录。[docs/compatibility.md](docs/compatibility.md) 包含三张表格，以及哪些内容可在 macOS、iOS 和 Linux 上运行的矩阵。

## 文档

- **API 文档。** `OpenJevCore`、`OpenJevEncoders`、`OpenJevDiffusionGemma`、`OpenJevLetterReadout` 和 `OpenJevServer` 的 DocC 目录：在应用中入门、请求和答案类型、实现后端、运行服务器以及配置参考。每当源代码变更时，Documentation 工作流都会构建它们并发布到 <https://algorythm-canada.github.io/OpenJevSwift/>。`make docs` 会在本地构建同样的站点（[docs/development.md](docs/development.md)）。
- **[docs/deployment.md](docs/deployment.md)**：在 Mac 上运行 `openjev serve`。
- **[docs/compatibility.md](docs/compatibility.md)**：哪些与上游完全相同，哪些在容差范围内，哪些不同。
- **[docs/credits.md](docs/credits.md)**：各个模型、它们的作者和许可证。
- **[docs/quality.md](docs/quality.md)** 和 **[docs/benchmarks.md](docs/benchmarks.md)**：与上游对比的答案质量，以及速度和内存。
- **[docs/README.md](docs/README.md)**：设计文档索引，从上游的工作原理到各项决策和一致性策略。
- **[CHANGELOG.md](CHANGELOG.md)**：每个发布版本以及版本策略。
- **[SECURITY.md](SECURITY.md)**：如何报告漏洞、代码会连接到哪里，以及它如何处理密钥。
- **[ADOPTERS.md](ADOPTERS.md)**：使用 OpenJevSwift 的组织；可以通过 pull request 添加你的组织。

## 参与贡献

欢迎任何人提交 bug 报告、兼容性报告、文档修正和 pull request。[CONTRIBUTING.md](CONTRIBUTING.md) 介绍了如何构建、测试以及提交 pull request。带有 [help wanted](https://github.com/Algorythm-Canada/OpenJevSwift/labels/help%20wanted) 标签的 issue 向所有人开放，[Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions) 是提问和交流想法的地方。

## 许可证与致谢

Apache-2.0，与上游 OpenJev 相同。从 mlx-vlm（MIT）移植的代码在每个文件的文件头中保留其版权声明；[THIRD_PARTY.md](THIRD_PARTY.md) 列出了所有引用的项目及其固定的修订版本。模型是他人的成果，保留各自的许可证：[docs/credits.md](docs/credits.md) 逐一致谢，并说明本项目从哪里获取它们的权重。本仓库不包含任何权重。
