# Instalação no macOS

Para usar o Vapor no macOS, você precisará do Swift 5.6 ou superior. O Swift e todas as suas dependências vêm incluídos com o Xcode.

## Instalar o Xcode

Instale o [Xcode](https://itunes.apple.com/us/app/xcode/id497799835?mt=12) na Mac App Store.

![Xcode na Mac App Store](../images/xcode-mac-app-store.png)

Depois que o Xcode for baixado, você deve abri-lo para completar a instalação. Isso pode levar algum tempo.

Verifique se a instalação foi bem-sucedida abrindo o Terminal e imprimindo a versão do Swift.

```sh
swift --version
```

Você deve ver as informações da versão do Swift impressas.

```sh
swift-driver version: 1.75.2 Apple Swift version 5.8 (swiftlang-5.8.0.124.2 clang-1403.0.22.11.100)
Target: arm64-apple-macosx13.0
```

O Vapor 4 requer o Swift 5.6 ou superior.

## Instalar o Toolbox

Agora que você tem o Swift instalado, vamos instalar o [Vapor Toolbox](https://github.com/vapor/toolbox). Esta ferramenta de linha de comando não é necessária para usar o Vapor, mas inclui utilitários úteis, como um criador de novos projetos.

O Toolbox é distribuído via Homebrew. Se você ainda não tem o Homebrew, visite <a href="https://brew.sh" target="_blank">brew.sh</a> para instruções de instalação.

```sh
vapor --help
```

Você deve ver uma lista de comandos disponíveis.

## Próximos Passos

Agora que você instalou o Swift e o Vapor Toolbox, crie seu primeiro aplicativo em [Primeiros Passos &rarr; Hello, world](../getting-started/hello-world.md).
