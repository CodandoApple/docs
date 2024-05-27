# Gerenciador de Pacotes Swift

O [Gerenciador de Pacotes Swift](https://swift.org/package-manager/) (SPM) é usado para construir o código-fonte do seu projeto e suas dependências. Como o Vapor depende muito do SPM, é uma boa ideia entender os conceitos básicos de como ele funciona.

O SPM é semelhante ao Cocoapods, Ruby gems e NPM. Você pode usar o SPM a partir da linha de comando com comandos como `swift build` e `swift test` ou com IDEs compatíveis. No entanto, ao contrário de alguns outros gerenciadores de pacotes, não há um índice central de pacotes para o SPM. O SPM, em vez disso, utiliza URLs para repositórios Git e gerencia versões de dependências usando [tags do Git](https://git-scm.com/book/en/v2/Git-Basics-Tagging).

## Manifesto do Pacote

O primeiro lugar que o SPM verifica em seu projeto é o manifesto do pacote. Isso sempre deve estar localizado no diretório raiz do seu projeto e nomeado `Package.swift`.

Veja este exemplo de manifesto do pacote.

```swift
// swift-tools-version:5.8
import PackageDescription

let package = Package(
    name: "MyApp",
    platforms: [
       .macOS(.v12)
    ],
    dependencies: [
        .package(url: "https://github.com/vapor/vapor.git", from: "4.76.0"),
    ],
    targets: [
        .executableTarget(
            name: "App",
            dependencies: [
                .product(name: "Vapor", package: "vapor")
            ]
        ),
        .testTarget(name: "AppTests", dependencies: [
            .target(name: "App"),
            .product(name: "XCTVapor", package: "vapor"),
        ])
    ]
)
```

Cada parte do manifesto é explicada nas seções seguintes.

### Versão das Ferramentas

A primeira linha de um manifesto de pacote indica a versão das ferramentas Swift necessárias. Isso especifica a versão mínima do Swift que o pacote suporta. A API de descrição do Pacote também pode mudar entre versões do Swift, então esta linha garante que o Swift saberá como analisar seu manifesto.

### Nome do Pacote

O primeiro argumento para `Package` é o nome do pacote. Se o pacote for público, você deve usar o último segmento da URL do repositório Git como o nome.

### Plataformas

O array `platforms` especifica quais plataformas este pacote suporta. Ao especificar `.macOS(.v12)`, este pacote requer macOS 12 ou posterior. Quando o Xcode carrega este projeto, ele automaticamente define a versão mínima de implantação para macOS 12, para que você possa usar todas as APIs disponíveis.

### Dependências

Dependências são outros pacotes SPM nos quais seu pacote depende. Todas as aplicações Vapor dependem do pacote Vapor, mas você pode adicionar quantas outras dependências desejar.

No exemplo acima, você pode ver [vapor/vapor](https://github.com/vapor/vapor) versão 4.76.0 ou posterior como uma dependência deste pacote. Quando você adiciona uma dependência ao seu pacote, você deve indicar quais [targets](#targets) dependem dos módulos recém-disponíveis.

### Targets

Targets são todos os módulos, executáveis e testes que seu pacote contém. A maioria dos aplicativos Vapor terá dois targets, embora você possa adicionar quantos quiser para organizar seu código. Cada target declara de quais módulos depende. Você deve adicionar nomes de módulos aqui para poder importá-los em seu código. Um target pode depender de outros targets em seu projeto ou de quaisquer módulos expostos por pacotes que você adicionou ao array de [dependências principais](#dependencies).

## Estrutura de Pastas

Abaixo está a estrutura de pastas típica para um pacote SPM.

```
.
├── Sources
│   └── App
│       └── (Source code)
├── Tests
│   └── AppTests
└── Package.swift
```

Cada `.target` ou `.executableTarget` corresponde a uma pasta na pasta `Sources`.
Cada `.testTarget` corresponde a uma pasta na pasta `Tests`.

## Package.resolved

A primeira vez que você construir seu projeto, o SPM criará um arquivo `Package.resolved` que armazena a versão de cada dependência. Na próxima vez que você construir seu projeto, essas mesmas versões serão usadas, mesmo que haja versões mais recentes disponíveis.

Para atualizar suas dependências, execute `swift package update`.

## Xcode

Se você estiver usando o Xcode 11 ou superior, as alterações em dependências, targets, produtos, etc., ocorrerão automaticamente sempre que o arquivo `Package.swift` for modificado.

Se você quiser atualizar para as últimas versões das dependências, use Arquivo &rarr; Pacotes Swift &rarr; Atualizar Para as Últimas Versões de Pacote Swift.

Você também pode querer adicionar o arquivo `.swiftpm` ao seu `.gitignore`. É aqui que o Xcode armazenará sua configuração do projeto Xcode.
