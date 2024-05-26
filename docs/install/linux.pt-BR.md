# Instalação no Linux

Para usar o Vapor, você precisará do Swift 5.7 ou superior. Isso pode ser instalado usando a ferramenta de linha de comando [Swiftly](https://swift-server.github.io/swiftly/) fornecida pelo Swift Server Workgroup (recomendado), ou as toolchains disponíveis em [Swift.org](https://swift.org/download/).

## Distribuições e Versões Suportadas

O Vapor suporta as mesmas versões de distribuições Linux que o Swift 5.7 ou versões mais recentes suportam. Por favor, consulte a [página oficial de suporte](https://www.swift.org/platform-support/) para encontrar informações atualizadas sobre quais sistemas operacionais são oficialmente suportados.

Distribuições Linux não oficialmente suportadas também podem rodar o Swift compilando o código fonte, mas o Vapor não pode garantir a estabilidade. Saiba mais sobre como compilar o Swift a partir do [repositório Swift](https://github.com/apple/swift#getting-started).

## Instalar o Swift

### Instalação automatizada usando a ferramenta de linha de comando Swiftly (recomendado)

Visite o [site do Swiftly](https://swift-server.github.io/swiftly/) para instruções sobre como instalar o Swiftly e o Swift no Linux. Depois disso, instale o Swift com o seguinte comando:

#### Uso básico

```sh
$ swiftly install latest

Fetching the latest stable Swift release...
Installing Swift 5.9.1
Downloaded 488.5 MiB of 488.5 MiB
Extracting toolchain...
Swift 5.9.1 installed successfully!

$ swift --version

Swift version 5.9.1 (swift-5.9.1-RELEASE)
Target: x86_64-unknown-linux-gnu
```

### Instalação manual com a toolchain

Visite o guia [Usando Downloads](https://swift.org/download/#using-downloads) do Swift.org para instruções sobre como instalar o Swift no Linux.

### Fedora

Os usuários do Fedora podem simplesmente usar o seguinte comando para instalar o Swift:

```sh
sudo dnf install swift-lang
```

Se você está usando o Fedora 35, precisará adicionar o EPEL 8 para obter o Swift 5.7 ou versões mais recentes.

## Docker

Você também pode usar as imagens oficiais do Docker do Swift, que vêm com o compilador pré-instalado. Saiba mais em [Docker Hub do Swift](https://hub.docker.com/_/swift).

## Instalar o Toolbox

Agora que você instalou o Swift, vamos instalar o [Vapor Toolbox](https://github.com/vapor/toolbox). Esta ferramenta de linha de comando não é necessária para usar o Vapor, mas inclui utilitários úteis.

No Linux, você precisará compilar o toolbox a partir do código fonte. Veja as <a href="https://github.com/vapor/toolbox/releases" target="_blank">versões</a> do toolbox no GitHub para encontrar a versão mais recente.

```sh
git clone https://github.com/vapor/toolbox.git
cd toolbox
git checkout <desired version>
make install
```

Double check the installation was successful by printing help.

```sh
vapor --help
```

Você deve ver uma lista de comandos disponíveis.

## Próximos Passos

Depois de instalar o Swift, crie seu primeiro aplicativo em [Primeiros Passos &rarr; Hello, world](../getting-started/hello-world.md).
