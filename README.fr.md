[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift – Documentation française

> Le README anglais reste la référence en cas de divergence.

OpenJevSwift fournit des décisions typées à partir des probabilités d'un modèle d'IA, plutôt que du texte libre. Il fonctionne sur les Mac Apple silicon via une API HTTP compatible Jev, ou dans une application iPhone et Mac. Les questions peuvent être de type oui/non (`noul`), choix (`choice`) ou score (`score`).

## Voir la démonstration

L'application de triage évalue pendant la saisie d'un message client son urgence, l'équipe destinataire et l'intensité du mécontentement. Chaque option affiche sa probabilité.

![TriageDemo](docs/assets/triage-demo.gif)

[Examples/TriageDemo](Examples/TriageDemo) · [Demo setup (English)](README.md#try-the-demo-app)

## Démarrage rapide

Prérequis : Mac Apple silicon, macOS 15 ou plus récent et Xcode 27 avec Metal Toolchain. Avec Xcode 26.4–26.6, utiliser `--build-system swiftbuild`. Les fichiers Verdict (~310 Mo) sont téléchargés au premier lancement et vérifiés par SHA-256. Le serveur écoute sur `127.0.0.1:8080`. `openjev decide` peut être exécuté sans serveur.

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

## Essayer l'application iPhone

[Examples/TriageDemo/README.md](Examples/TriageDemo/README.md) · Xcode 26.4+, iOS 18+, iPhone or simulator. First launch downloads and SHA-256 verifies the Verdict files; subsequent launches work offline.

## Configuration requise

| Backend | Model | Mac | Memory |
|---|---|---|---|
| `verdict` | `verdict-1.4`, 151M parameters, Core ML | Apple silicon, macOS 15 or later | 1.6 GB with the functions one-question reads load, 2.8 GB with all six |
| `laya` | `laya-1.0`, 421M parameters, Core ML | Apple silicon, macOS 15 or later | 4.7 GB with the functions one-question reads load, 8.9 GB with all eight and up to 9.7 GB at peak; with `OPENJEV_ENCODER_FUNCTIONS=2`, 2.1 GB for one-question reads and up to 4.4 GB at peak |
| `mlx` | `openjev-0.1`, DiffusionGemma 26B-A4B, 4-bit, MLX | Apple silicon | about 17 GB to load, the vision tower's 1.06 GiB included (D-054); 17.3 GiB in service with short prompts, measured before the tower loaded, and up to about 3.6 GB more for cached long prompts; 32 GB or more recommended |
| `jevk5` | `jevk5-0.2`, JevK5 (Qwen3.5-4B), 8-bit, MLX | Apple silicon | 6.0 GB once loaded, up to 11.0 GB in service with `OPENJEV_MLX_CACHE_LIMIT_GB=4`; 3.6 and 8.9 GB with the 4-bit conversion |

La construction exige Xcode 26.4 ou plus récent. `OpenJevCore` nécessite macOS 14 ou iOS 17 ; Verdict et Laya nécessitent macOS 15 ou iOS 18. JevK5 se compile pour iOS, mais n'a pas encore été exécuté sur iPhone. Linux construit le cœur, le serveur et l'outil pour les tests, sans moteur de modèle.

## État du projet

La version `0.1.0` est la première utilisable comme dépendance. Les jalons 0 à 5 couvrent le cœur, le serveur, MLX, les images, les extensions et la génération de texte avec `think` et chat. Les jalons 6 et 7 couvrent Verdict, Laya, JevK5, les évaluations JevBench et la calibration. Restent ouverts : CLM, différé jusqu'à demande explicite, et l'optimisation des lectures DiffusionGemma (#100 à #102). [Implementation plan](docs/08-implementation-plan.md) · [Changelog](CHANGELOG.md).

## Compatibilité et évaluations

Les prompts, validations, erreurs et réponses ont été confrontés à OpenJev en amont, sauf différences consignées dans [les tableaux de compatibilité](docs/compatibility.md). DiffusionGemma : 63 lectures de référence, 91,7 % des meilleurs labels identiques ; 139 sur 140 lorsque l'écart entre les deux premiers scores est d'au moins 0,5 ; comparaison sur 333 éléments JevBench et TypeSafe. Verdict et Laya : meilleur résultat identique sur 666 éléments. JevK5 : meilleur résultat identique sur 230 des 231 éléments, et nombres de jetons identiques. Il ne s'agit pas d'une équivalence numérique bit à bit générale.

[Compatibility matrices](docs/compatibility.md) · [Quality report](docs/quality.md) · [Benchmarks](docs/benchmarks.md).

## Documentation

Les cinq modules DocC sont `OpenJevCore`, `OpenJevEncoders`, `OpenJevDiffusionGemma`, `OpenJevLetterReadout` et `OpenJevServer`. Le workflow publie les documents ; `make docs` les construit localement.

- [DocC API](https://algorythm-canada.github.io/OpenJevSwift/)
- [Development](docs/development.md) · [Deployment](docs/deployment.md)
- [Compatibility](docs/compatibility.md) · [Credits](docs/credits.md)
- [Quality](docs/quality.md) · [Benchmarks](docs/benchmarks.md)
- [Documentation index](docs/README.md) · [Changelog](CHANGELOG.md) · [Security](SECURITY.md) · [Adopters](ADOPTERS.md)

## Contribuer

Les rapports de bogues, les rapports de compatibilité, les corrections de documentation et les demandes de tirage sont bienvenus. Consultez les tâches `help wanted` et [CONTRIBUTING.md](CONTRIBUTING.md). [GitHub Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions).

## Licence et crédits

Projet indépendant sous licence Apache-2.0. Aucune affiliation ou approbation par TypeSafe AI, Google DeepMind, NVIDIA ou les auteurs des modèles. Les poids des modèles ne sont pas hébergés ici ; leurs auteurs et licences figurent dans [les crédits](docs/credits.md) et [THIRD_PARTY.md](THIRD_PARTY.md).
