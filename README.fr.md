[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift – Présentation en français

> Cette page est une introduction traduite et non une traduction intégrale. Le [README anglais](README.md) reste la documentation de référence, notamment pour les limitations, les versions et les résultats des tests.

OpenJevSwift est une implémentation native en Swift d'[OpenJev](https://github.com/razorback16/openjev). Elle permet de poser des questions typées à un modèle d'IA et d'obtenir des probabilités structurées, au lieu de texte libre : réponse oui/non (`noul`), choix entre plusieurs options (`choice`) et évaluation sur une échelle (`score`).

Le serveur compatible avec l'API de Jev fonctionne sur les Mac Apple silicon. Les bibliothèques peuvent également être intégrées à des applications iPhone et Mac. Les modèles pris en charge comprennent Verdict et Laya avec Core ML, ainsi que DiffusionGemma et JevK5 avec MLX.

## Voir la démonstration

![Application de triage sur simulateur iPhone](docs/assets/triage-demo.gif)

L'[application de démonstration](Examples/TriageDemo) analyse un message de client pendant la saisie : urgence, équipe destinataire et degré de mécontentement. Pour les prérequis et les instructions de lancement, consultez la [démonstration dans le README anglais](README.md#try-the-demo-app).

## Démarrage rapide sur Mac

Il faut un Mac Apple silicon, macOS 15 ou ultérieur et Xcode 27 avec le composant Metal Toolchain. Pour les autres versions de Xcode, consultez les [instructions officielles](README.md#quick-start).

```bash
xcodebuild -downloadComponent MetalToolchain
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

Au premier lancement, le modèle nécessaire est téléchargé et ses empreintes SHA-256 sont vérifiées. Consultez le [README anglais](README.md#quick-start) pour l'exemple de requête API.

## Documentation et contributions

- [Documentation API](https://algorythm-canada.github.io/OpenJevSwift/)
- [Configuration et déploiement](docs/deployment.md)
- [Compatibilité et limites](docs/compatibility.md)
- [Contribuer](CONTRIBUTING.md) et [discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions)

Le projet est indépendant et n'est ni affilié à TypeSafe AI, ni approuvé par les auteurs des modèles qu'il utilise. Les modèles conservent leurs licences et leurs crédits respectifs. Voir [les crédits](docs/credits.md).
