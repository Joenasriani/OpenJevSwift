[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift – التوثيق باللغة العربية

> يبقى ملف README الإنجليزي المرجع عند اختلاف التفاصيل.

يقدّم OpenJevSwift قرارات ذكاء اصطناعي محددة النوع اعتمادًا على احتمالات النموذج بدلًا من توليد نص حر. يعمل على أجهزة Mac بمعالجات Apple silicon عبر واجهة HTTP متوافقة مع Jev، أو مباشرة داخل تطبيقات iPhone وMac. تشمل أنواع الأسئلة نعم/لا (`noul`)، اختيارًا بين بدائل (`choice`)، ودرجة ضمن مقياس (`score`).

## مشاهدة العرض

يقيّم تطبيق فرز رسائل العملاء على iPhone، أثناء كتابة الرسالة، مدى الإلحاح والفريق المسؤول ودرجة الانزعاج. ويعرض احتمال كل خيار.

![TriageDemo](docs/assets/triage-demo.gif)

[Examples/TriageDemo](Examples/TriageDemo) · [Demo setup (English)](README.md#try-the-demo-app)

## البدء السريع

المتطلبات: جهاز Mac بمعالج Apple silicon ونظام macOS 15 أو أحدث وXcode 27 مع Metal Toolchain. مع Xcode من 26.4 إلى 26.6 استخدم `--build-system swiftbuild`. عند التشغيل الأول تُنزّل ملفات Verdict بحجم يقارب 310 ميغابايت ويُتحقق من بصماتها عبر SHA-256. يستمع الخادم على `127.0.0.1:8080`، ويمكن تشغيل `openjev decide` دون خادم.

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

## تجربة تطبيق iPhone

[Examples/TriageDemo/README.md](Examples/TriageDemo/README.md) · Xcode 26.4+, iOS 18+, iPhone or simulator. First launch downloads and SHA-256 verifies the Verdict files; subsequent launches work offline.

## المتطلبات

| Backend | Model | Mac | Memory |
|---|---|---|---|
| `verdict` | `verdict-1.4`, 151M parameters, Core ML | Apple silicon, macOS 15 or later | 1.6 GB with the functions one-question reads load, 2.8 GB with all six |
| `laya` | `laya-1.0`, 421M parameters, Core ML | Apple silicon, macOS 15 or later | 4.7 GB with the functions one-question reads load, 8.9 GB with all eight and up to 9.7 GB at peak; with `OPENJEV_ENCODER_FUNCTIONS=2`, 2.1 GB for one-question reads and up to 4.4 GB at peak |
| `mlx` | `openjev-0.1`, DiffusionGemma 26B-A4B, 4-bit, MLX | Apple silicon | about 17 GB to load, the vision tower's 1.06 GiB included (D-054); 17.3 GiB in service with short prompts, measured before the tower loaded, and up to about 3.6 GB more for cached long prompts; 32 GB or more recommended |
| `jevk5` | `jevk5-0.2`, JevK5 (Qwen3.5-4B), 8-bit, MLX | Apple silicon | 6.0 GB once loaded, up to 11.0 GB in service with `OPENJEV_MLX_CACHE_LIMIT_GB=4`; 3.6 and 8.9 GB with the 4-bit conversion |

يتطلب البناء Xcode 26.4 أو أحدث. تدعم `OpenJevCore` ‏macOS 14 وiOS 17؛ وتتطلب Verdict وLaya ‏macOS 15 وiOS 18. يمكن بناء JevK5 لنظام iOS لكن لم يُختبر تشغيله على iPhone بعد. يستطيع Linux بناء النواة والخادم وأداة الأوامر للاختبارات دون محرك نماذج.

## حالة المشروع

الإصدار `0.1.0` هو أول إصدار مناسب للتبعيات البرمجية. تشمل المراحل 0–5 النواة والخادم وMLX والصور والامتدادات وتوليد النص بما فيه `think` والمحادثة. تشمل المرحلتان 6 و7 ‏Verdict وLaya وJevK5 ومقارنات JevBench والمعايرة. المتبقي: نموذج CLM المؤجل إلى حين الطلب، وتحسين سرعة قراءات DiffusionGemma (المهام #100 إلى #102). [Implementation plan](docs/08-implementation-plan.md) · [Changelog](CHANGELOG.md).

## التوافق ونتائج الاختبارات

تمت مقارنة القوالب والتحقق والأخطاء والإجابات بتنفيذ OpenJev الأصلي باستثناء الاختلافات الموثقة. DiffusionGemma: ‏63 قراءة مرجعية؛ توافق الخيار الأعلى في 91.7% من المواضع وفي 139 من 140 حين يكون الفرق بين أعلى قيمتين 0.5 على الأقل؛ مقارنة مع 333 عنصرًا من JevBench وTypeSafe. Verdict وLaya: تطابق أعلى إجابة في 666 عنصرًا. JevK5: تطابق أعلى إجابة في 230 من 231 عنصرًا مع تطابق أعداد الرموز. لا يعني ذلك تطابق جميع الاحتمالات بتًا مقابل بت.

[Compatibility matrices](docs/compatibility.md) · [Quality report](docs/quality.md) · [Benchmarks](docs/benchmarks.md).

## التوثيق

تشمل وثائق DocC خمس وحدات: `OpenJevCore` و`OpenJevEncoders` و`OpenJevDiffusionGemma` و`OpenJevLetterReadout` و`OpenJevServer`. ينشر مسار العمل الوثائق، ويمكن بناؤها محليًا عبر `make docs`.

- [DocC API](https://algorythm-canada.github.io/OpenJevSwift/)
- [Development](docs/development.md) · [Deployment](docs/deployment.md)
- [Compatibility](docs/compatibility.md) · [Credits](docs/credits.md)
- [Quality](docs/quality.md) · [Benchmarks](docs/benchmarks.md)
- [Documentation index](docs/README.md) · [Changelog](CHANGELOG.md) · [Security](SECURITY.md) · [Adopters](ADOPTERS.md)

## المساهمة

نرحب بتقارير الأخطاء واختبارات التوافق وتصحيحات التوثيق وطلبات السحب. راجع المهام المعلّمة `help wanted` و[CONTRIBUTING.md](CONTRIBUTING.md). [GitHub Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions).

## الترخيص والاعتمادات

المشروع مستقل برخصة Apache-2.0، ولا ينتمي إلى TypeSafe AI أو Google DeepMind أو NVIDIA أو مؤلفي النماذج ولا يحظى بتأييدهم. لا تُحفظ أوزان النماذج هنا. راجع [الاعتمادات والتراخيص](docs/credits.md) و[THIRD_PARTY.md](THIRD_PARTY.md).
