# Olá, mundo

Este guia o levará passo a passo pela criação de um novo projeto Vapor, construindo-o e executando o servidor.

Se você ainda não instalou o Swift ou o Vapor Toolbox, confira a seção de instalação.

- [Instalação &rarr; macOS](../install/macos.md)
- [Instalação &rarr; Linux](../install/linux.md)

## Novo Projeto

O primeiro passo é criar um novo projeto Vapor no seu computador. Abra o seu terminal e use o comando de novo projeto do Toolbox. Isso criará uma nova pasta no diretório atual contendo o projeto.

```sh
vapor new hello -n
```

!!! tip
    A flag `-n` lhe dá um template básico, respondendo automaticamente não a todas as perguntas.

!!! tip
    Você também pode obter o template mais recente do GitHub sem o Vapor Toolbox, clonando o [repositório de templates](https://github.com/vapor/template-bare)

!!! tip
    O Vapor e o template agora usam `async`/`await` por padrão.
    Se você não puder atualizar para o macOS 12 e/ou precisar continuar usando `EventLoopFuture`s,
    use a flag `--branch macos10-15`.

Depois que o comando terminar, mude para a pasta recém-criada:

```sh
cd hello
```

## Construir & Executar

### Xcode

Primeiro, abra o projeto no Xcode:

```sh
open Package.swift
```

Ele começará automaticamente a baixar as dependências do Gerenciador de Pacotes Swift. Isso pode levar algum tempo na primeira vez que você abrir um projeto. Quando a resolução de dependência estiver completa, o Xcode irá preencher os esquemas disponíveis.

No topo da janela, à direita dos botões Play e Stop, clique no nome do seu projeto para selecionar o Esquema do projeto e selecione um alvo de execução apropriado — provavelmente, "Meu Mac". Clique no botão play para construir e executar seu projeto.

Você deve ver o Console aparecer na parte inferior da janela do Xcode.

```sh
[ INFO ] Server starting on http://127.0.0.1:8080
```

### Linux

No Linux e em outros sistemas operacionais (e até no macOS, se você não quiser usar o Xcode), você pode editar o projeto no seu editor de escolha, como Vim ou VSCode. Veja os [Guias do Servidor Swift](https://github.com/swift-server/guides/blob/main/docs/setup-and-ide-alternatives.md) para detalhes atualizados sobre a configuração de outros IDEs.

Para construir e executar seu projeto, no Terminal execute:

```sh
swift run
```

Isso construirá e executará o projeto. A primeira vez que você executar isso levará algum tempo para buscar e resolver as dependências. Uma vez executando, você deve ver o seguinte no seu console:

```sh
[ INFO ] Server starting on http://127.0.0.1:8080
```

## Visite o Localhost

Abra seu navegador web e visite <a href="http://localhost:8080/hello" target="_blank">localhost:8080/hello</a> ou <a href="http://127.0.0.1:8080" target="_blank">http://127.0.0.1:8080</a>

Você deve ver a seguinte página.

```html
Hello, world!
```

Parabéns por criar, construir e executar seu primeiro aplicativo Vapor! 🎉
