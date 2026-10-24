[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift – Deutsche Dokumentation

> Bei Abweichungen ist die englische README maßgeblich.

OpenJevSwift liefert typisierte KI-Entscheidungen aus Modellwahrscheinlichkeiten statt frei erzeugtem Text. Es läuft auf Apple-Silicon-Macs über eine Jev-kompatible HTTP-API oder direkt in iPhone- und Mac-Apps. Unterstützte Fragetypen sind Ja/Nein (`noul`), Auswahl (`choice`) und Bewertung (`score`).

## Demo ansehen

Die iPhone-Triage-App prüft Kundennachrichten während der Eingabe auf Dringlichkeit, das zuständige Team und den Grad der Unzufriedenheit. Für jede Option zeigt sie Wahrscheinlichkeiten.

![TriageDemo](docs/assets/triage-demo.gif)

[Examples/TriageDemo](Examples/TriageDemo) · [Demo setup (English)](README.md#try-the-demo-app)

## Schnellstart

Voraussetzungen: Apple-Silicon-Mac mit macOS 15 oder neuer sowie Xcode 27 mit Metal Toolchain. Mit Xcode 26.4–26.6 ist `--build-system swiftbuild` erforderlich. Beim ersten Start werden Verdict-Dateien (~310 MB) heruntergeladen und mit SHA-256 geprüft. Der Server lauscht auf `127.0.0.1:8080`. `openjev decide` benötigt keinen laufenden Server.

```bash
xcodebuild -downloadComponent MetalToolchain
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

```bash
curl -s localhost:8080/v1/systemone -H 'content-type: application/json' \
    -d '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}'
```

```json
{"model":"verdict-1.4","answers":{"urgent":{"type":"noul","noul":0.5910340547561646}},"usage":{"input_tokens":45,"output_tokens":0}}
```

```bash
echo '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}' | .build/release/openjev decide --backend verdict
```

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

## iPhone-Demo ausprobieren

[Examples/TriageDemo/README.md](Examples/TriageDemo/README.md) · Xcode 26.4+, iOS 18+, iPhone or simulator. First launch downloads and SHA-256 verifies the Verdict files; subsequent launches work offline.

## Anforderungen

| Backend | Model | Mac | Memory |
|---|---|---|---|
| `verdict` | `verdict-1.4`, 151M parameters, Core ML | Apple silicon, macOS 15 or later | 1.6 GB with the functions one-question reads load, 2.8 GB with all six |
| `laya` | `laya-1.0`, 421M parameters, Core ML | Apple silicon, macOS 15 or later | 4.7 GB with the functions one-question reads load, 8.9 GB with all eight and up to 9.7 GB at peak; with `OPENJEV_ENCODER_FUNCTIONS=2`, 2.1 GB for one-question reads and up to 4.4 GB at peak |
| `mlx` | `openjev-0.1`, DiffusionGemma 26B-A4B, 4-bit, MLX | Apple silicon | about 17 GB to load, the vision tower's 1.06 GiB included (D-054); 17.3 GiB in service with short prompts, measured before the tower loaded, and up to about 3.6 GB more for cached long prompts; 32 GB or more recommended |
| `jevk5` | `jevk5-0.2`, JevK5 (Qwen3.5-4B), 8-bit, MLX | Apple silicon | 6.0 GB once loaded, up to 11.0 GB in service with `OPENJEV_MLX_CACHE_LIMIT_GB=4`; 3.6 and 8.9 GB with the 4-bit conversion |

Für den Build wird Xcode 26.4 oder neuer benötigt. `OpenJevCore` unterstützt macOS 14 und iOS 17; Verdict und Laya benötigen macOS 15 und iOS 18. JevK5 kann für iOS gebaut werden, wurde aber noch nicht auf einem iPhone ausgeführt. Linux baut Core, Server und CLI ohne Modell-Backend für Tests.

## Projektstand

Release `0.1.0` ist die erste Paketversion. Meilensteine 0 bis 5 umfassen Kern, Server, MLX, Bilder, Erweiterungen sowie Texterzeugung mit `think` und Chat. Meilensteine 6 und 7 umfassen Verdict, Laya, JevK5, JevBench-Vergleich und Kalibrierung. Offen sind CLM (bis zu einer begründeten Anfrage zurückgestellt) sowie die Beschleunigung von DiffusionGemma (#100 bis #102). [Implementation plan](docs/08-implementation-plan.md) · [Changelog](CHANGELOG.md).

## Kompatibilität und Ergebnisse

Prompts, Prüfungen, Fehler und Antworten wurden abgesehen von dokumentierten Unterschieden mit upstream OpenJev abgeglichen. DiffusionGemma: 63 Referenz-Lesevorgänge, führendes Label auf 91,7 % der Positionen gleich, auf 139 von 140 bei mindestens 0,5 Abstand zwischen den beiden höchsten Scores; Vergleich auf 333 JevBench- und TypeSafe-Elementen. Verdict und Laya: führende Antwort für alle 666 Elemente gleich. JevK5: führende Antwort für 230 von 231 Elementen gleich, mit identischen Tokenzahlen. Modellwahrscheinlichkeiten sind nicht durchgehend bitgenau identisch.

[Compatibility matrices](docs/compatibility.md) · [Quality report](docs/quality.md) · [Benchmarks](docs/benchmarks.md).

## Dokumentation

Die DocC-Dokumentation umfasst `OpenJevCore`, `OpenJevEncoders`, `OpenJevDiffusionGemma`, `OpenJevLetterReadout` und `OpenJevServer`. Der Workflow veröffentlicht die Dokumentation; `make docs` baut sie lokal.

- [DocC API](https://algorythm-canada.github.io/OpenJevSwift/)
- [Development](docs/development.md) · [Deployment](docs/deployment.md)
- [Compatibility](docs/compatibility.md) · [Credits](docs/credits.md)
- [Quality](docs/quality.md) · [Benchmarks](docs/benchmarks.md)
- [Documentation index](docs/README.md) · [Changelog](CHANGELOG.md) · [Security](SECURITY.md) · [Adopters](ADOPTERS.md)

## Mitwirken

Fehlerberichte, Kompatibilitätsprüfungen, Dokumentationskorrekturen und Pull Requests sind willkommen. Siehe `help wanted` und [CONTRIBUTING.md](CONTRIBUTING.md). [GitHub Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions).

## Lizenz und Quellen

Unabhängiges Projekt unter Apache-2.0, ohne Zugehörigkeit oder Unterstützung durch TypeSafe AI, Google DeepMind, NVIDIA oder die Modellautoren. Die Modellgewichte werden nicht hier gespeichert. Lizenzen und Quellen stehen unter [Credits](docs/credits.md) und [THIRD_PARTY.md](THIRD_PARTY.md).
