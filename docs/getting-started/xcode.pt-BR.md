# Xcode

Esta página aborda algumas dicas e truques para usar o Xcode. Se você usa um ambiente de desenvolvimento diferente, pode ignorar esta seção.

## Diretório de Trabalho Personalizado

Por padrão, o Xcode executará seu projeto a partir da pasta _DerivedData_. Esta pasta não é a mesma que a pasta raiz do seu projeto (onde seu arquivo _Package.swift_ está). Isso significa que o Vapor não será capaz de encontrar arquivos e pastas como _.env_ ou _Public_.

Você pode perceber que isso está acontecendo se vir o seguinte aviso ao executar seu aplicativo.

```fish
[ WARNING ] No custom working directory set for this scheme, using /path/to/DerivedData/project-abcdef/Build/
```

Para corrigir isso, defina um diretório de trabalho personalizado no esquema do Xcode para o seu projeto.

Primeiro, edite o esquema do seu projeto clicando no seletor de esquemas ao lado dos botões de play e stop.

![Área do Esquema do Xcode](../images/xcode-scheme-area.png)

Selecione _Edit Scheme(Editar Esquema)..._ no menu suspenso.

![Menu do Esquema do Xcode](../images/xcode-scheme-menu.png)

No editor de esquemas, escolha a ação _App_ e a aba _Opções_. Marque _Usar diretório de trabalho personalizado_ e insira o caminho para a pasta raiz do seu projeto.

![Opções do Esquema do Xcode](../images/xcode-scheme-options.png)

Você pode obter o caminho completo para a pasta raiz do seu projeto executando `pwd` de uma janela do terminal aberta lá.

```fish
# verifique se estamos na pasta do projeto vapor
vapor --version
# obtenha o caminho para esta pasta
pwd
```

Você deve ver uma saída semelhante à seguinte.

```
framework: 4.x.x
toolbox: 18.x.x
/path/to/project
```
