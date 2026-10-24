[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift – مقدمة باللغة العربية

> هذه الصفحة مقدمة مترجمة وليست ترجمة كاملة للوثائق. يبقى [ملف README باللغة الإنجليزية](README.md) المرجع المعتمد، خصوصًا بشأن القيود والإصدارات ونتائج الاختبارات.

OpenJevSwift هو تنفيذ أصلي بلغة Swift لمشروع [OpenJev](https://github.com/razorback16/openjev). يتيح طرح أسئلة محددة النوع على نموذج ذكاء اصطناعي والحصول على احتمالات منظّمة بدلًا من نص حر: نعم أو لا (`noul`)، الاختيار بين خيارات (`choice`)، أو التقييم ضمن مقياس (`score`).

يعمل خادم HTTP المتوافق مع واجهة Jev على أجهزة Mac بمعالجات Apple silicon. ويمكن دمج المكتبات داخل تطبيقات iPhone وMac. تشمل النماذج المدعومة Verdict وLaya باستخدام Core ML، وDiffusionGemma وJevK5 باستخدام MLX.

## مشاهدة العرض التجريبي

![تطبيق فرز رسائل العملاء على محاكي iPhone](docs/assets/triage-demo.gif)

يعرض [التطبيق التجريبي](Examples/TriageDemo) كيفية تحليل رسالة عميل أثناء كتابتها، بما يشمل مدى الإلحاح والفريق المناسب ودرجة انزعاج العميل. راجع [تعليمات تشغيل العرض بالإنجليزية](README.md#try-the-demo-app) لمعرفة المتطلبات والتفاصيل.

## البدء السريع على Mac

يتطلب المثال جهاز Mac بمعالج Apple silicon ونظام macOS 15 أو أحدث، مع Xcode 27 ومكوّن Metal Toolchain. للاطلاع على إصدارات Xcode الأخرى راجع [الدليل الإنجليزي](README.md#quick-start).

```bash
xcodebuild -downloadComponent MetalToolchain
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

عند التشغيل الأول تُنزّل ملفات النموذج المطلوبة ويُتحقّق من سلامتها باستخدام بصمات SHA-256. يتضمن [الدليل الإنجليزي](README.md#quick-start) مثالًا على طلب API.

## التوثيق والمساهمة

- [توثيق API](https://algorythm-canada.github.io/OpenJevSwift/)
- [التشغيل والنشر](docs/deployment.md)
- [التوافق والقيود](docs/compatibility.md)
- [دليل المساهمة](CONTRIBUTING.md) و[النقاشات](https://github.com/Algorythm-Canada/OpenJevSwift/discussions)

المشروع مستقل وغير تابع لشركة TypeSafe AI ولا يحظى بتأييد مؤلفي النماذج المستخدمة. تخضع النماذج لتراخيصها ونسبها إلى أصحابها. انظر [صفحة الاعتمادات](docs/credits.md).
