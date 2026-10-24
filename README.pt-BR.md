[English](README.md) | [简体中文](README.zh-CN.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português](README.pt-BR.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [العربية](README.ar.md)

# OpenJevSwift

[![CI](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/ci.yml) [![Fixtures](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/fixtures.yml) [![Documentation](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml/badge.svg?branch=main)](https://github.com/Algorythm-Canada/OpenJevSwift/actions/workflows/docs.yml)

> Se esta tradução divergir do [README em inglês](README.md), a versão em inglês prevalece.

Faça perguntas tipadas a um modelo sobre um trecho de texto (sim ou não, qual de várias opções, ou quanto em uma
escala) e receba probabilidades em vez de texto gerado. O OpenJevSwift as responde localmente: no seu Mac, por trás de
uma API HTTP compatível com Jev, ou dentro do seu próprio app para iPhone ou Mac. Ele foi feito para desenvolvedores
Swift que querem que essas decisões sejam tomadas no dispositivo e para usuários de Mac que querem a API do OpenJev em
um único binário nativo em vez de um ambiente Python. O [app de demonstração](#experimente-o-app-de-demonstração) o
mostra respondendo enquanto você digita.

Uma implementação nativa em Swift do [OpenJev](https://github.com/razorback16/openjev), o servidor de decisões
"System One" aberto e compatível com Jev. Envie a ele um estado e perguntas tipadas (`noul`, `choice`, `score`); ele lê
cada resposta das probabilidades de um modelo em vez de gerar texto, então uma resposta não pode sair do esquema: a de
um noul é a probabilidade de sim, e a de um choice ou de um score, a probabilidade de cada opção, com uma confiança. Um
único binário `openjev` serve o DiffusionGemma 26B-A4B ou o JevK5 no MLX, ou o Verdict ou o Laya no Core ML, em um Mac,
com exatamente o mesmo formato de requisições e respostas do upstream, por isso os SDKs da TypeSafe funcionam com ele
sem alterações; as mesmas bibliotecas respondem requisições dentro de um app para iPhone ou Mac.

O OpenJevSwift é um projeto independente. Ele não é afiliado à TypeSafe AI (criadora do Jev), ao Google DeepMind ou à
NVIDIA (DiffusionGemma), nem aos autores dos outros modelos que serve, nem conta com o endosso deles. Jev, TypeSafe,
Gemma e outros nomes são propriedade de seus respectivos titulares.

## Início rápido

Em um Mac com Apple silicon, macOS 15 ou posterior e Xcode 27 (o primeiro comando instala, uma única vez, o Metal
Toolchain do Xcode, como explicado em
[docs/development.md](docs/development.md#xcode-27-needs-the-metal-toolchain)):

```bash
xcodebuild -downloadComponent MetalToolchain
git clone https://github.com/Algorythm-Canada/OpenJevSwift.git
cd OpenJevSwift
swift build -c release --product openjev
.build/release/openjev serve --backend verdict
```

Com as versões 26.4 a 26.6 do Xcode, adicione `--build-system swiftbuild` ao `swift build`; caso contrário, os backends
`mlx` e `jevk5` ficam sem os shaders Metal do MLX.

A primeira inicialização baixa o pacote Core ML convertido do Verdict, o tokenizador e o calibrador (cerca de 310 MB)
em Application Support e verifica o SHA-256 de cada arquivo; o servidor escuta em `127.0.0.1:8080` assim que o modelo
termina de carregar e aquecer. Em outro terminal:

```bash
curl -s localhost:8080/v1/systemone -H 'content-type: application/json' \
    -d '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}'
```

```json
{"model":"verdict-1.4","answers":{"urgent":{"type":"noul","noul":0.5910340547561646}},"usage":{"input_tokens":45,"output_tokens":0}}
```

`openjev decide` responde a uma requisição sem servidor e imprime os bytes que o servidor envia:

```bash
echo '{"model":"jev-latest","state":"The deploy failed twice and the site is down.","questions":{"urgent":{"type":"noul","instructions":"Is this urgent?"}}}' | .build/release/openjev decide --backend verdict
```

Ctrl-C encerra o servidor de forma ordenada. [docs/deployment.md](docs/deployment.md) cobre as configurações, um job do
launchd, os logs e os códigos de saída.

Um app depende de uma release e vincula o `OpenJevCore` e o módulo de cada backend que carrega: `OpenJevEncoders` para
o Verdict e o Laya, `OpenJevDiffusionGemma` para o DiffusionGemma e `OpenJevLetterReadout` para o JevK5. O artigo do
DocC
[Making decisions in an app](https://algorythm-canada.github.io/OpenJevSwift/documentation/openjevcore/gettingstarted/)
carrega os mesmos modelos dentro de um app:

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

## Experimente o app de demonstração

![A demo de triagem respondendo a uma reclamação de cobrança enquanto é digitada, em um simulador de iPhone](docs/assets/triage-demo.gif)

[Examples/TriageDemo](Examples/TriageDemo) é um app para iPhone que responde, no dispositivo, ao exemplo do README do
upstream: se a mensagem de um cliente precisa de resposta dentro de uma hora (`noul`), qual equipe deve cuidar dela
(`choice`) e o quanto o cliente está chateado (`score`), com a probabilidade de cada opção como uma barra, enquanto você
digita. Ele precisa do Xcode 26.4 ou posterior e de um iPhone ou um simulador com iOS 18 ou posterior:

1. Feche qualquer janela do Xcode que esteja com o pacote OpenJevSwift aberto: o Xcode só permite que uma janela use
   um pacote local.
2. Abra `Examples/TriageDemo/TriageDemo.xcodeproj`.
3. Para executar em um iPhone, escolha sua equipe em Signing & Capabilities; o simulador não precisa de nenhuma.
4. Execute o esquema `TriageDemo`.

A primeira inicialização baixa o Verdict pelo próprio armazenamento da biblioteca, cerca de 310 MB da release
openjev-models e do Hugging Face, e verifica o SHA-256 de cada arquivo; as inicializações seguintes funcionam offline.
[Examples/TriageDemo/README.md](Examples/TriageDemo/README.md) tem os detalhes e os testes.

## Requisitos

| Backend | Modelo | Mac | Memória |
|---|---|---|---|
| `verdict` | `verdict-1.4`, 151M parâmetros, Core ML | Apple silicon, macOS 15 ou posterior | 1.6 GB com as funções que as leituras de uma só pergunta carregam, 2.8 GB com todas as seis |
| `laya` | `laya-1.0`, 421M parâmetros, Core ML | Apple silicon, macOS 15 ou posterior | 4.7 GB com as funções que as leituras de uma só pergunta carregam, 8.9 GB com todas as oito e até 9.7 GB no pico; com `OPENJEV_ENCODER_FUNCTIONS=2`, 2.1 GB para leituras de uma só pergunta e até 4.4 GB no pico |
| `mlx` | `openjev-0.1`, DiffusionGemma 26B-A4B, 4-bit, MLX | Apple silicon | cerca de 17 GB para carregar, incluindo os 1.06 GiB da torre de visão (D-054); 17.3 GiB em serviço com prompts curtos, medidos antes de a torre ser carregada, e até cerca de 3.6 GB a mais para prompts longos em cache; recomenda-se 32 GB ou mais |
| `jevk5` | `jevk5-0.2`, JevK5 (Qwen3.5-4B), 8-bit, MLX | Apple silicon | 6.0 GB depois de carregado, até 11.0 GB em serviço com `OPENJEV_MLX_CACHE_LIMIT_GB=4`; 3.6 e 8.9 GB com a conversão de 4-bit |

Para compilar, é preciso ter o Xcode 26.4 ou posterior. O backend `mlx` também precisa dos shaders Metal do MLX, que o
Swift Build compila com o Metal Toolchain: o Swift Build é o padrão no Xcode 27, e com o Xcode 26 passa-se
`--build-system swiftbuild`. Em um app, o `OpenJevCore` roda no macOS 14 e no iOS 17 ou posterior, os backends Verdict
e Laya no macOS 15 e no iOS 18 ou posterior, e o JevK5 em Apple silicon (ele compila para iOS; ainda não foi executado
em um iPhone). No Linux, são compilados o núcleo, o servidor e a ferramenta `openjev` para os testes, sem nenhum
backend.

## Status

A release 0.1.0 é a primeira versão da qual um pacote pode depender; [CHANGELOG.md](CHANGELOG.md) lista o que ela
inclui. Concluído:

- **Marcos 0 a 5**, com todas as issues de trabalho fechadas: as fundações, o núcleo do mecanismo de decisão, as
  leituras do DiffusionGemma no MLX, o servidor HTTP compatível com Jev com a ferramenta `openjev`, as extensões de
  leitura e as imagens, e a geração de texto com `think` e chat completions (#50 a #53).
- **DiffusionGemma, verificado no seu checkpoint:** `steps`, `samples` e `sequential` de ponta a ponta (#43, #44 e
  #45), e as leituras de imagens, que coincidem bit a bit com as do upstream nos kernels do oráculo (#46 a #48).
- **Do marco 6:** Verdict e Laya, e o JevK5 (#55), que dá a resposta principal publicada pelo seu autor em 230 dos 231
  itens do JevBench (D-052).
- **Do marco 7:** a comparação do JevBench com o upstream e o relatório de calibração do DiffusionGemma (#61 e #62).

Pendente:

- O modelo CLM, adiado até que alguém o peça
  ([D-011](docs/06-decisions.md#d-011-encoder-models-core-ml-for-verdict-and-laya-jevk5-first-among-the-extra-models)).
  Se você precisar dele, abra uma [issue](https://github.com/Algorythm-Canada/OpenJevSwift/issues) ou uma publicação em
  [Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions) com o seu caso de uso.
- Leituras mais rápidas do DiffusionGemma: #100, #101 e #102.

[docs/08-implementation-plan.md](docs/08-implementation-plan.md) tem os marcos e o índice de issues.

## Compatibilidade

Fora as diferenças registradas, tudo até as probabilidades do modelo é idêntico byte a byte ao upstream: os prompts, os
templates de resposta, os canvas e as sementes, a validação de requisições, os corpos de erro, os cabeçalhos e a
listagem de `/v1/models`, tudo verificado com fixtures que o próprio código do upstream gera. As probabilidades
concordam dentro de limites medidos:

- **DiffusionGemma:** dentro dos limites das decisões D-014 e D-048 nas 63 leituras do oráculo (o rótulo principal em
  91.7% dos slots, e em 139 dos 140 em que os dois primeiros rótulos do mlx-vlm estão separados por pelo menos 0.5), e
  entre os dois servidores em 333 itens do JevBench e da TypeSafe.
- **Verdict e Laya:** a resposta principal do upstream em todos os 666 itens.
- **JevK5, na sua conversão de 8-bit:** a resposta principal publicada pelo seu autor em 230 dos 231 itens do JevBench,
  com as mesmas contagens de tokens em todos eles.

As diferenças, entre elas um parser de JSON mais rigoroso, o 503 para qualquer falha do backend e os recursos ainda não
implementados, têm cada uma um registro de decisão. [docs/compatibility.md](docs/compatibility.md) tem as três tabelas
e a matriz do que roda no macOS, no iOS e no Linux.

## Documentação

- **Documentação da API.** Os catálogos DocC de `OpenJevCore`, `OpenJevEncoders`, `OpenJevDiffusionGemma`,
  `OpenJevLetterReadout` e `OpenJevServer`: primeiros passos em um app, os tipos de requisição e de resposta, como
  implementar um backend, como executar o servidor e a referência de configuração. O workflow Documentation os gera
  sempre que os arquivos-fonte mudam e os publica em <https://algorythm-canada.github.io/OpenJevSwift/>. `make docs`
  gera o mesmo site localmente ([docs/development.md](docs/development.md)).
- **[docs/deployment.md](docs/deployment.md)**: como executar `openjev serve` em um Mac.
- **[docs/compatibility.md](docs/compatibility.md)**: o que é idêntico ao upstream, o que está dentro da tolerância e o
  que é diferente.
- **[docs/credits.md](docs/credits.md)**: os modelos, seus autores e suas licenças.
- **[docs/quality.md](docs/quality.md)** e **[docs/benchmarks.md](docs/benchmarks.md)**: a qualidade das respostas em
  comparação com o upstream, e a velocidade e a memória.
- **[docs/README.md](docs/README.md)**: o índice dos documentos de design, de como o upstream funciona até as decisões
  e a estratégia de conformidade.
- **[CHANGELOG.md](CHANGELOG.md)**: cada release e a política de versionamento.
- **[SECURITY.md](SECURITY.md)**: como relatar uma vulnerabilidade, a que o código se conecta e como ele lida com
  chaves.
- **[ADOPTERS.md](ADOPTERS.md)**: organizações que usam o OpenJevSwift; adicione a sua com um pull request.

## Como contribuir

Qualquer pessoa é bem-vinda para enviar relatos de bugs, relatos de compatibilidade, correções na documentação e pull
requests. [CONTRIBUTING.md](CONTRIBUTING.md) explica como compilar, testar e abrir um pull request. As issues com o
rótulo [help wanted](https://github.com/Algorythm-Canada/OpenJevSwift/labels/help%20wanted) estão abertas a qualquer
pessoa, e [Discussions](https://github.com/Algorythm-Canada/OpenJevSwift/discussions) é o lugar para perguntas e ideias.

## Licença e créditos

Apache-2.0, a mesma do OpenJev upstream. O código portado do mlx-vlm (MIT) mantém o aviso de copyright no cabeçalho de
cada arquivo; [THIRD_PARTY.md](THIRD_PARTY.md) lista cada projeto referenciado na sua revisão fixada. Os modelos são
trabalho de outras pessoas e mantêm suas próprias licenças: [docs/credits.md](docs/credits.md) dá o crédito a cada um e
indica de onde este projeto obtém os pesos. Nenhum peso faz parte deste repositório.
