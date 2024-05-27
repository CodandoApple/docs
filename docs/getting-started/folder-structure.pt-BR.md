# Estrutura de Pastas

Agora que você criou, construiu e executou seu primeiro aplicativo Vapor, vamos aproveitar um momento para familiarizá-lo com a estrutura de pastas do Vapor. A estrutura é baseada na estrutura de pastas do [SPM](spm.md), então, se você já trabalhou com SPM antes, ela deve ser familiar.

```
.
├── Public
├── Sources
│   ├── App
│   │   ├── Controllers
│   │   ├── Migrations
│   │   ├── Models
│   │   ├── configure.swift
│   │   ├── entrypoint.swift
│   │   └── routes.swift
│
├── Tests
│   └── AppTests
└── Package.swift
```

As seções abaixo explicam cada parte da estrutura de pastas com mais detalhes.

## Public

Esta pasta contém quaisquer arquivos públicos que serão servidos pelo seu aplicativo se o `FileMiddleware` estiver habilitado. Isso geralmente inclui imagens, folhas de estilo e scripts de navegador. Por exemplo, uma solicitação para `localhost:8080/favicon.ico` verificará se `Public/favicon.ico` existe e o retornará.

Você precisará habilitar o `FileMiddleware` no seu arquivo `configure.swift` antes que o Vapor possa servir arquivos públicos.

```swift
// Serve arquivos do diretório `Public/`
let fileMiddleware = FileMiddleware(
    publicDirectory: app.directory.publicDirectory
)
app.middleware.use(fileMiddleware)
```

## Sources

Esta pasta contém todos os arquivos fonte Swift do seu projeto.
A pasta de nível superior, `App`, reflete o módulo do seu pacote,
como declarado no manifesto do [SwiftPM](spm.md).

### App

Aqui é onde toda a lógica da sua aplicação deve ser colocada.

#### Controllers

Controllers são uma excelente maneira de agrupar lógica de aplicação. A maioria dos controllers possui várias funções que aceitam uma solicitação e retornam algum tipo de resposta.

#### Migrations

A pasta de migrations é onde vão suas migrações de banco de dados, se você estiver usando Fluent.

#### Models

A pasta de models é um ótimo lugar para armazenar seus `structs` de `Content` ou `Model`s do Fluent.

#### configure.swift

Este arquivo contém a função `configure(_:)`. Este método é chamado por `entrypoint.swift` para configurar a `Application` recém-criada. Aqui é onde você deve registrar serviços como rotas, bancos de dados, provedores e mais.

#### entrypoint.swift

Este arquivo contém o ponto de entrada `@main` para a aplicação que configura e executa sua aplicação Vapor.

#### routes.swift

Este arquivo contém a função `routes(_:)`. Este método é chamado perto do fim de `configure(_:)` para registrar rotas na sua `Application`.

## Tests

Cada módulo não executável em sua pasta `Sources` pode ter uma pasta correspondente em `Tests`. Isso contém código construído sobre o módulo `XCTest` para testar seu pacote. Os testes podem ser executados usando `swift test` no terminal ou pressionando ⌘+U no Xcode.

### AppTests

Esta pasta contém os testes unitários para o código no seu módulo `App`.

## Package.swift

Finalmente, o manifesto do pacote do [SPM](spm.md).
