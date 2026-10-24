[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift

[![CI](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml) [![Fixtures](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml) [![Documentation](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml)

> 이 번역이 [영어 README](README.md)와 다를 경우 영어 버전이 우선합니다.

텍스트에 대해 모델에 타입이 있는 질문(예 또는 아니요, 여러 선택지 중 어느 것인지, 척도에서 어느 정도인지)을 하고, 생성된 텍스트 대신 확률을 돌려받습니다. OpenJevSwift는 이러한 질문에 로컬에서 답합니다. Mac에서 Jev 호환 HTTP API 뒤에서 동작하거나, 직접 만든 iPhone 또는 Mac 앱 안에서 동작합니다. 이러한 의사결정을 기기에서 처리하려는 Swift 개발자와, Python 환경 대신 하나의 네이티브 바이너리로 OpenJev API를 사용하려는 Mac 사용자를 위한 프로젝트입니다. [데모 앱](#데모-앱-사용해-보기)에서 입력하는 동안 답하는 모습을 볼 수 있습니다.

[OpenJev](https://github.com/razorback16/openjev)의 네이티브 Swift 구현입니다. OpenJev는 개방형 Jev 호환 "System One" 의사결정 서버입니다. 상태와 타입이 있는 질문(`noul`, `choice`, `score`)을 보내면, 텍스트를 생성하는 대신 모델의 확률에서 모든 답을 읽어 내므로 답이 스키마를 벗어날 수 없습니다. noul의 답은 예일 확률이고, choice나 score의 답은 모든 선택지의 확률과 신뢰도입니다. Mac에서는 하나의 `openjev` 바이너리가 MLX 기반의 DiffusionGemma 26B-A4B 또는 JevK5, 혹은 Core ML 기반의 Verdict 또는 Laya를 서빙합니다. 요청·응답 형식이 업스트림과 정확히 같으므로 TypeSafe의 SDK를 수정 없이 그대로 사용할 수 있습니다. 같은 라이브러리가 iPhone 또는 Mac 앱 안에서도 요청에 답합니다.

OpenJevSwift는 독립 프로젝트입니다. TypeSafe AI(Jev 제작사), Google DeepMind 또는 NVIDIA(DiffusionGemma), 그리고 이 프로젝트가 서빙하는 다른 모델의 제작자와 제휴 관계가 없으며, 이들의 보증을 받지도 않았습니다. Jev, TypeSafe, Gemma 및 기타 명칭은 각 소유자의 자산입니다.

## 빠른 시작

macOS 15 이상과 Xcode 27이 설치된 Apple 실리콘 Mac에서 실행합니다(첫 번째 명령은 Xcode의 Metal Toolchain을 한 번만 설치하는 것으로,
[docs/development.md](docs/development.md#xcode-27-needs-the-metal-toolchain)에서 설명합니다).

```bash
xcodebuild -downloadComponent MetalToolchain
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

Xcode 26.4부터 26.6까지는 `swift build` 명령에 `--build-system swiftbuild` 옵션을 추가하세요. 그렇지 않으면 `mlx` 및 `jevk5` 백엔드에
MLX의 Metal 셰이더가 빠집니다.

처음 시작할 때 Verdict의 변환된 Core ML 패키지, 토크나이저, 캘리브레이터(약 310 MB)를 Application Support에 내려받고 각 파일의 SHA-256을
확인합니다. 모델이 로드되고 워밍업이 끝나면 서버는 `127.0.0.1:8080` 주소에서 수신 대기합니다. 다른 터미널에서 실행합니다.

```bash
curl -s localhost:8080/v1/systemone -H 'content-type: application/json' \
    -d '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}'
```

```json
{"model":"verdict-1.4","answers":{"urgent":{"type":"noul","noul":0.5910340547561646}},"usage":{"input_tokens":45,"output_tokens":0}}
```

`openjev decide` 명령은 서버 없이 요청 하나에 답하고, 서버가 보내는 바이트를 출력합니다.

```bash
echo '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}' | .build/release/openjev decide --backend verdict
```

Ctrl-C를 누르면 서버가 정상적으로 종료됩니다. [docs/deployment.md](docs/deployment.md)에서 설정, launchd 작업, 로그, 종료 상태를 다룹니다.

앱은 릴리스에 의존하며 `OpenJevCore` 모듈과, 로드하는 각 백엔드의 모듈을 링크합니다. Verdict와 Laya에는 `OpenJevEncoders` 모듈,
DiffusionGemma에는 `OpenJevDiffusionGemma` 모듈, JevK5에는 `OpenJevLetterReadout` 모듈이 필요합니다. DocC 문서
[Making decisions in an app](https://algorythm-canada.github.io/OpenJevSwift/documentation/openjevcore/gettingstarted/)에서는 같은 모델을
앱 안에서 로드합니다.

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

## 데모 앱 사용해 보기

![iPhone 시뮬레이터에서 청구 관련 불만이 입력되는 동안 답하는 트리아지 데모](docs/assets/triage-demo.gif)

[Examples/TriageDemo](Examples/TriageDemo)는 업스트림 README의 예제에 기기에서 답하는 iPhone 앱입니다. 고객 메시지에 한 시간 안에 답장이
필요한지(`noul`), 어느 팀이 처리해야 하는지(`choice`), 고객이 얼마나 화가 났는지(`score`)를 입력하는 동안 답하며, 모든 선택지의 확률을 막대로
보여 줍니다. Xcode 26.4 이상과 iOS 18 이상의 iPhone 또는 시뮬레이터가 필요합니다.

1. OpenJevSwift 패키지가 열려 있는 Xcode 창을 모두 닫습니다. Xcode에서는 한 창만 로컬 패키지를 사용할 수 있습니다.
2. `Examples/TriageDemo/TriageDemo.xcodeproj` 파일을 엽니다.
3. iPhone에서 실행하려면 Signing & Capabilities에서 팀을 선택합니다. 시뮬레이터에는 필요하지 않습니다.
4. `TriageDemo` 스킴을 실행합니다.

처음 실행할 때 라이브러리 자체 스토어를 통해 Verdict를 내려받습니다. openjev-models 릴리스와 Hugging Face에서 약 310 MB를 받고 각 파일의
SHA-256을 확인하며, 이후 실행은 오프라인에서 동작합니다. 자세한 내용과 테스트는
[Examples/TriageDemo/README.md](Examples/TriageDemo/README.md)에 있습니다.

## 요구 사항

| 백엔드 | 모델 | Mac | 메모리 |
|---|---|---|---|
| `verdict` | `verdict-1.4`, 151M 파라미터, Core ML | Apple 실리콘, macOS 15 이상 | 단일 질문 읽기가 로드하는 함수로 1.6 GB, 6개 모두로 2.8 GB |
| `laya` | `laya-1.0`, 421M 파라미터, Core ML | Apple 실리콘, macOS 15 이상 | 단일 질문 읽기가 로드하는 함수로 4.7 GB, 8개 모두로 8.9 GB, 피크 시 최대 9.7 GB. `OPENJEV_ENCODER_FUNCTIONS=2` 설정 시 단일 질문 읽기에 2.1 GB, 피크 시 최대 4.4 GB |
| `mlx` | `openjev-0.1`, DiffusionGemma 26B-A4B, 4-bit, MLX | Apple 실리콘 | 로드에 약 17 GB(비전 타워의 1.06 GiB 포함, D-054). 짧은 프롬프트로 서비스 중 17.3 GiB(타워 로드 전 측정), 캐시된 긴 프롬프트에 최대 약 3.6 GB 추가. 32 GB 이상 권장 |
| `jevk5` | `jevk5-0.2`, JevK5(Qwen3.5-4B), 8-bit, MLX | Apple 실리콘 | 로드 후 6.0 GB, `OPENJEV_MLX_CACHE_LIMIT_GB=4` 설정 시 서비스 중 최대 11.0 GB. 4-bit 변환 버전은 3.6 GB와 8.9 GB |

빌드에는 Xcode 26.4 이상이 필요합니다. `mlx` 백엔드에는 MLX의 Metal 셰이더도 필요하며, Swift Build가 Metal Toolchain으로 이 셰이더를
컴파일합니다. Xcode 27에서는 Swift Build가 기본값이고, Xcode 26에서는 `--build-system swiftbuild` 옵션을 지정합니다. 앱에서
`OpenJevCore` 모듈은 macOS 14 및 iOS 17 이상, Verdict와 Laya 백엔드는 macOS 15 및 iOS 18 이상, JevK5는 Apple 실리콘에서 실행됩니다(iOS용으로
빌드는 되지만 아직 iPhone에서 실행된 적은 없습니다). Linux에서는 테스트를 위해 코어, 서버, `openjev` 도구를 빌드하며, 백엔드는 포함하지
않습니다.

## 현황

릴리스 0.1.0은 패키지가 의존할 수 있는 첫 버전이며, 포함된 내용은 [CHANGELOG.md](CHANGELOG.md)에 나와 있습니다. 완료된 항목:

- **마일스톤 0부터 5까지**(그 안의 모든 작업 이슈 완료): 기반, 의사결정 엔진 코어, MLX 기반 DiffusionGemma 읽기, `openjev` 도구를 포함한
  Jev 호환 HTTP 서버, 읽기 확장과 이미지, `think` 및 chat completions를 지원하는 텍스트 생성(#50~#53).
- **DiffusionGemma, 체크포인트에서 검증됨:** `steps`, `samples`, `sequential` 엔드 투 엔드(#43, #44, #45), 그리고 오라클의 커널에서
  업스트림과 비트 단위로 일치하는 이미지 읽기(#46~#48).
- **마일스톤 6에서:** Verdict와 Laya, 그리고 JevK5(#55). JevK5는 JevBench의 231개 항목 중 230개에서 제작자가 공개한 최상위 답을
  냅니다(D-052).
- **마일스톤 7에서:** 업스트림과의 JevBench 비교와 DiffusionGemma의 캘리브레이션 보고서(#61, #62).

아직 남은 항목:

- CLM 모델. 누군가 요청할 때까지 보류합니다
  ([D-011](docs/06-decisions.md#d-011-encoder-models-core-ml-for-verdict-and-laya-jevk5-first-among-the-extra-models)).
  필요하다면 사용 사례를 담아 [이슈](https://github.com/Algorythm-Canada/OpenJevSwift/issues)를 열거나
  [Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions)에 글을 올려 주세요.
- 더 빠른 DiffusionGemma 읽기: #100, #101, #102.

마일스톤과 이슈 색인은 [docs/08-implementation-plan.md](docs/08-implementation-plan.md)에 있습니다.

## 호환성

기록된 차이를 제외하면 모델의 확률에 이르기까지 모든 것이 업스트림과 바이트 단위로 같습니다. 프롬프트, 답 템플릿, 캔버스와 시드, 요청 검증,
오류 본문, 헤더, `/v1/models` 목록 모두 업스트림 자체 코드가 작성한 픽스처와 대조해 확인했습니다. 확률은 다음과 같이 측정된 범위 안에서
일치합니다.

- **DiffusionGemma:** 63회의 오라클 읽기에서 결정 D-014 및 D-048의 범위 안에 있으며(슬롯의 91.7%에서 최상위 레이블이 일치하고, mlx-vlm의
  상위 두 값이 0.5 이상 차이 나는 140개 중 139개에서 일치), 333개의 JevBench 및 TypeSafe 항목에서 두 서버 사이에서도 범위 안에 있습니다.
- **Verdict와 Laya:** 666개 항목 모두에서 업스트림의 최상위 답과 일치합니다.
- **JevK5, 8-bit 변환 버전:** JevBench의 231개 항목 중 230개에서 제작자가 공개한 최상위 답을 내며, 모든 항목에서 토큰 수도 같습니다.

더 엄격한 JSON 파서, 모든 백엔드 실패에 대한 503, 아직 구현되지 않은 기능 등 각 차이에는 결정 기록이 있습니다.
[docs/compatibility.md](docs/compatibility.md)에는 세 개의 표와, macOS, iOS, Linux에서 무엇이 실행되는지 보여 주는 매트릭스가 있습니다.

## 문서

- **API 문서.** `OpenJevCore`, `OpenJevEncoders`, `OpenJevDiffusionGemma`, `OpenJevLetterReadout`, `OpenJevServer` 모듈의 DocC 카탈로그:
  앱에서 시작하기, 요청 및 답 타입, 백엔드 구현, 서버 실행, 구성 참조. Documentation 워크플로가 소스가 바뀔 때마다 이를 빌드해
  <https://algorythm-canada.github.io/OpenJevSwift/>에 게시합니다. `make docs` 명령으로 같은 사이트를 로컬에서 빌드할 수 있습니다
  ([docs/development.md](docs/development.md)).
- **[docs/deployment.md](docs/deployment.md)**: Mac에서 `openjev serve` 실행하기.
- **[docs/compatibility.md](docs/compatibility.md)**: 업스트림과 동일한 것, 허용 오차 안에 있는 것, 다른 것.
- **[docs/credits.md](docs/credits.md)**: 모델, 그 제작자와 라이선스.
- **[docs/quality.md](docs/quality.md)** 및 **[docs/benchmarks.md](docs/benchmarks.md)**: 업스트림 대비 답 품질, 그리고 속도와 메모리.
- **[docs/README.md](docs/README.md)**: 설계 문서 색인. 업스트림의 동작 방식부터 결정 사항과 적합성 전략까지 다룹니다.
- **[CHANGELOG.md](CHANGELOG.md)**: 각 릴리스와 버전 관리 정책.
- **[SECURITY.md](SECURITY.md)**: 취약점 보고 방법, 코드가 연결하는 대상, 키 처리 방식.
- **[ADOPTERS.md](ADOPTERS.md)**: OpenJevSwift를 사용하는 조직. pull request로 여러분의 조직을 추가하세요.

## 기여하기

버그 보고, 호환성 보고, 문서 수정, pull request는 누구든 환영합니다. [CONTRIBUTING.md](CONTRIBUTING.md)에서 빌드, 테스트, pull request
열기를 다룹니다. [help wanted](https://github.com/Algorythm-Canada/OpenJevSwift/labels/help%20wanted) 레이블이 붙은 이슈는 누구나 맡을 수
있으며, [Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions)는 질문과 아이디어를 위한 곳입니다.

## 라이선스 및 크레디트

Apache-2.0이며, 업스트림 OpenJev와 같습니다. mlx-vlm(MIT)에서 포팅한 코드는 각 파일 헤더에 저작권 고지를 유지합니다.
[THIRD_PARTY.md](THIRD_PARTY.md)에는 참조한 모든 프로젝트가 고정된 리비전과 함께 나열되어 있습니다. 모델은 다른 사람들의 작업물이며 각자의
라이선스를 따릅니다. [docs/credits.md](docs/credits.md)에서 각 모델의 크레디트와 이 프로젝트가 가중치를 가져오는 곳을 밝힙니다. 이 저장소에는
가중치가 포함되어 있지 않습니다.
