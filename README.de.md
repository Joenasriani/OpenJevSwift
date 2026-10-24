[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift – Deutschsprachiger Überblick

> Diese Seite ist eine übersetzte Einführung, keine vollständige Übersetzung. Maßgeblich ist die [englische README](README.md), insbesondere bei Einschränkungen, Versionsangaben und Testergebnissen.

OpenJevSwift ist eine native Swift-Implementierung von [OpenJev](https://github.com/razorback16/openjev). Die Software beantwortet typisierte KI-Fragen mit strukturierten Modellwahrscheinlichkeiten statt frei erzeugtem Text: Ja/Nein (`noul`), Auswahl zwischen Optionen (`choice`) und Bewertung auf einer Skala (`score`).

Ein Jev-kompatibler HTTP-Server läuft auf Apple-Silicon-Macs. Die Bibliotheken lassen sich außerdem in iPhone- und Mac-Apps integrieren. Zu den unterstützten Modellen gehören Verdict und Laya mit Core ML sowie DiffusionGemma und JevK5 mit MLX.

## Demo ansehen

![Triage-Demo im iPhone-Simulator](docs/assets/triage-demo.gif)

Die [Beispiel-App](Examples/TriageDemo) bewertet eine Kundennachricht während der Eingabe: Dringlichkeit, zuständiges Team und Grad der Unzufriedenheit. Anforderungen und Schritte zum Starten stehen im [englischen Demo-Abschnitt](README.md#try-the-demo-app).

## Schnellstart auf dem Mac

Voraussetzungen sind ein Apple-Silicon-Mac, macOS 15 oder neuer und Xcode 27 mit der Metal Toolchain. Hinweise zu anderen Xcode-Versionen stehen in der [englischen Anleitung](README.md#quick-start).

```bash
xcodebuild -downloadComponent MetalToolchain
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

Beim ersten Start werden die benötigten Modelldateien geladen und anhand von SHA-256-Prüfsummen überprüft. Ein API-Beispiel steht im [englischen Schnellstart](README.md#quick-start).

## Dokumentation und Mitarbeit

- [API-Dokumentation](https://algorythm-canada.github.io/OpenJevSwift/)
- [Betrieb und Bereitstellung](docs/deployment.md)
- [Kompatibilität und Einschränkungen](docs/compatibility.md)
- [Mitwirken](CONTRIBUTING.md) und [Diskussionen](https://github.com/Algorythm-Canada/OpenJevSwift/discussions)

Das Projekt ist unabhängig und weder mit TypeSafe AI verbunden noch von den Autoren der verwendeten Modelle unterstützt. Für die Modelle gelten ihre jeweiligen Lizenzen und Urheberangaben. Siehe [Quellen und Danksagungen](docs/credits.md).
