# Roteamento

Roteamento é o processo de encontrar o manipulador de requisições apropriado para uma requisição recebida. No coração do roteamento do Vapor está um roteador de nó trie de alto desempenho do [RoutingKit](https://github.com/vapor/routing-kit).

## Visão Geral

Para entender como o roteamento funciona no Vapor, você deve primeiro entender alguns conceitos básicos sobre requisições HTTP. Dê uma olhada no seguinte exemplo de requisição.

```http
GET /hello/vapor HTTP/1.1
host: vapor.codes
content-length: 0
```

Esta é uma simples requisição HTTP `GET` para a URL `/hello/vapor`. Este é o tipo de requisição HTTP que seu navegador faria se você o direcionasse para a seguinte URL.

```
http://vapor.codes/hello/vapor
```

### Método HTTP

A primeira parte da requisição é o método HTTP. `GET` é o método HTTP mais comum, mas existem vários que você usará frequentemente. Esses métodos HTTP são frequentemente associados com a semântica [CRUD](https://pt.wikipedia.org/wiki/CRUD) (Criar, Ler, Atualizar, Deletar).

| Método  | CRUD        |
|---------|-------------|
| `GET`   | Ler         |
| `POST`  | Criar       |
| `PUT`   | Substituir  |
| `PATCH` | Atualizar   |
| `DELETE`| Deletar     |

### Caminho da Requisição

Logo após o método HTTP está o URI da requisição. Isso consiste em um caminho que começa com `/` e uma string de consulta opcional após `?`. O método HTTP e o caminho são o que o Vapor usa para rotear as requisições.

Após o URI vem a versão HTTP seguida de zero ou mais cabeçalhos e, finalmente, um corpo. Como esta é uma requisição `GET`, ela não possui corpo.

### Métodos de Rotas

Vamos dar uma olhada em como essa requisição poderia ser tratada no Vapor.

```swift
app.get("hello", "vapor") { req in 
    return "Hello, vapor!"
}
```

Todos os métodos HTTP comuns estão disponíveis como métodos em `Application`. Eles aceitam um ou mais argumentos de string que representam o caminho da requisição, separados por `/`.

Observe que você também pode escrever isso usando `on` seguido pelo método.

```swift
app.on(.GET, "hello", "vapor") { ... }
```

Com esta rota registrada, a requisição HTTP do exemplo acima resultará na seguinte resposta HTTP.

```http
HTTP/1.1 200 OK
content-length: 13
content-type: text/plain; charset=utf-8

Hello, vapor!
```

### Parâmetros da Rota

Agora que encaminhamos com sucesso uma solicitação com base no método HTTP e no caminho, vamos tentar tornar o caminho dinâmico. Observe que o nome "vapor" está codificado tanto no caminho quanto na resposta. Vamos tornar isso dinâmico para que você possa visitar `/hello/<qualquer nome>` e obter uma resposta.

```swift
app.get("hello", ":name") { req -> String in
    let name = req.parameters.get("name")!
    return "Hello, \(name)!"
}
```

Ao usar um componente de caminho prefixado com `:`, indicamos ao roteador que este é um componente dinâmico. Qualquer string fornecida aqui agora corresponderá a esta rota. Podemos então usar `req.parameters` para acessar o valor da string.

Se você executar a solicitação de exemplo novamente, ainda receberá uma resposta que diz olá para vapor. No entanto, agora você pode incluir qualquer nome após `/hello/` e vê-lo incluído na resposta. Vamos tentar `/hello/swift`.

```http
GET /hello/swift HTTP/1.1
content-length: 0
```
```http
HTTP/1.1 200 OK
content-length: 13
content-type: text/plain; charset=utf-8

Hello, swift!
```

Agora que você entende os conceitos básicos, confira cada seção para aprender mais sobre parâmetros, grupos e mais.

## Rotas

Uma rota especifica um manipulador de requisições para um dado método HTTP e caminho URI. Ela também pode armazenar metadados adicionais.

### Métodos

Rotas podem ser registradas diretamente no seu `Application` usando vários auxiliares de método HTTP.

```swift
// responde a GET /foo/bar/baz
app.get("foo", "bar", "baz") { req in
	...
}
```

Os manipuladores de rota suportam o retorno de qualquer coisa que seja `ResponseEncodable`. Isso inclui `Content`, uma closure `async`, e quaisquer `EventLoopFuture`s onde o valor futuro seja `ResponseEncodable`.

Você pode especificar o tipo de retorno de uma rota usando `-> T` antes de `in`. Isso pode ser útil em situações onde o compilador não consegue determinar o tipo de retorno.

```swift
app.get("foo") { req -> String in
	return "bar"
}
```

Estes são os métodos auxiliares de rota suportados:

- `get`
- `post`
- `patch`
- `put`
- `delete`

Além dos auxiliares de método HTTP, existe uma função `on` que aceita o método HTTP como um parâmetro de entrada.

```swift
// responde a OPTIONS /foo/bar/baz
app.on(.OPTIONS, "foo", "bar", "baz") { req in
	...
}
```

### Componente de Caminho

Cada método de registro de rota aceita uma lista variável de `PathComponent`. Este tipo é expresso por literal de string e possui quatro casos:

- Constante (`foo`)
- Parâmetro (`:foo`)
- Qualquer coisa (`*`)
- Captura tudo (`**`)

#### Constante

Este é um componente de rota estático. Apenas solicitações com uma string exatamente correspondente nesta posição serão permitidas.

```swift
// responde a GET /foo/bar/baz
app.get("foo", "bar", "baz") { req in
	...
}
```

#### Parâmetro

Este é um componente de rota dinâmico. Qualquer string nesta posição será permitida. Um componente de caminho de parâmetro é especificado com um prefixo `:`. A string que segue o `:` será usada como o nome do parâmetro. Você pode usar o nome para buscar posteriormente o valor do parâmetro na requisição.

```swift
// responde a GET /foo/bar/baz
// responde a GET /foo/qux/baz
// ...
app.get("foo", ":bar", "baz") { req in
	...
}
```

#### Qualquer coisa

Isso é muito semelhante ao parâmetro, exceto que o valor é descartado. Este componente de caminho é especificado apenas como `*`.

```swift
// responde a GET /foo/bar/baz
// responde a GET /foo/qux/baz
// ...
app.get("foo", "*", "baz") { req in
	...
}
```

#### Captura tudo

Este é um componente de rota dinâmico que corresponde a um ou mais componentes. É especificado usando apenas `**`. Qualquer string nesta posição ou posições posteriores será correspondida na requisição.

```swift
// responde a GET /foo/bar
// responde a GET /foo/bar/baz
// ...
app.get("foo", "**") { req in 
    ...
}
```

### Parâmetros

Ao usar um componente de caminho de parâmetro (prefixado com `:`), o valor do URI nessa posição será armazenado em `req.parameters`. Você pode usar o nome do componente de caminho para acessar o valor.

```swift
// responde a GET /hello/foo
// responde a GET /hello/bar
// ...
app.get("hello", ":name") { req -> String in
    let name = req.parameters.get("name")!
    return "Hello, \(name)!"
}
```

!!! tip
    Podemos ter certeza de que `req.parameters.get` nunca retornará `nil` aqui, pois nosso caminho da rota inclui `:name`. No entanto, se você estiver acessando parâmetros de rota em middleware ou em código acionado por várias rotas, você vai querer lidar com a possibilidade de `nil`.

!!! tip
    Se você quiser recuperar parâmetros de consulta da URL, por exemplo, `/hello/?name=foo`, você precisa usar as APIs de Conteúdo do Vapor para lidar com dados codificados por URL na string de consulta da URL. Veja a [referência `Content`](content.md) para mais detalhes.

`req.parameters.get` também suporta a conversão do parâmetro para tipos `LosslessStringConvertible` automaticamente.


```swift
// responde a GET /number/42
// responde a GET /number/1337
// ...
app.get("number", ":x") { req -> String in 
	guard let int = req.parameters.get("x", as: Int.self) else {
		throw Abort(.badRequest)
	}
	return "\(int) is a great number"
}
```

Os valores do URI correspondidos por Captura Tudo (`**`) serão armazenados em `req.parameters` como `[String]`. Você pode usar `req.parameters.getCatchall` para acessar esses componentes.

```swift
// responde a GET /hello/foo
// responde a GET /hello/foo/bar
// ...
app.get("hello", "**") { req -> String in
    let name = req.parameters.getCatchall().joined(separator: " ")
    return "Hello, \(name)!"
}
```

### Streaming de Corpo

Ao registrar uma rota usando o método `on`, você pode especificar como o corpo da requisição deve ser tratado. Por padrão, os corpos das requisições são coletados na memória antes de chamar seu manipulador. Isso é útil, pois permite que a decodificação do conteúdo da requisição seja síncrona, mesmo que sua aplicação leia requisições de entrada de forma assíncrona.

Por padrão, o Vapor limitará a coleta do corpo em streaming a 16KB de tamanho. Você pode configurar isso usando `app.routes`.

```swift
// Aumenta o limite de coleta do body em streaming para 500kb
app.routes.defaultMaxBodySize = "500kb"
```

Se um corpo em streaming que está sendo coletado exceder o limite configurado, um erro `413 Payload Too Large` será lançado.

Para configurar a estratégia de coleta do corpo da requisição para uma rota individual, use o parâmetro `body`.

```swift
// Coleta corpos em streaming (até 1MB de tamanho) antes de chamar esta rota.
app.on(.POST, "listings", body: .collect(maxSize: "1mb")) { req in
    // Handle request. 
}
```

Se um `maxSize` for passado para `collect`, ele substituirá o padrão da aplicação para essa rota. Para usar o padrão da aplicação, omita o argumento `maxSize`.

Para requisições grandes, como uploads de arquivos, coletar o corpo da requisição em um buffer pode potencialmente sobrecarregar a memória do sistema. Para evitar que o corpo da requisição seja coletado, use a estratégia `stream`.

```swift
// O body da requisição não será coletado em um buffer.
app.on(.POST, "upload", body: .stream) { req in
    ...
}
```

Quando o corpo da requisição é transmitido, `req.body.data` será `nil`. Você deve usar `req.body.drain` para tratar cada pedaço conforme ele é enviado para sua rota.

### Roteamento Insensível a Maiúsculas e Minúsculas

O comportamento padrão para roteamento é sensível a maiúsculas e minúsculas e preserva o caso. Componentes de caminho `Constant` podem ser tratados de maneira insensível a maiúsculas e minúsculas, mas preservando o caso, para fins de roteamento; para habilitar esse comportamento, configure antes da inicialização da aplicação:

```swift
app.routes.caseInsensitive = true
```

Nenhuma alteração é feita na requisição original; os manipuladores de rota receberão os componentes do caminho da requisição sem modificação.

### Visualizando Rotas

Você pode acessar as rotas da sua aplicação utilizando o serviço `Routes` ou usando `app.routes`.

```swift
print(app.routes.all) // [Route]
```

O Vapor também vem com um comando `routes` que imprime todas as rotas disponíveis em uma tabela formatada em ASCII.

```sh
$ swift run App routes
+--------+----------------+
| GET    | /              |
+--------+----------------+
| GET    | /hello         |
+--------+----------------+
| GET    | /todos         |
+--------+----------------+
| POST   | /todos         |
+--------+----------------+
| DELETE | /todos/:todoID |
+--------+----------------+
```

### Metadados

Todos os métodos de registro de rota retornam a `Route` criada. Isso permite que você adicione metadados ao dicionário `userInfo` da rota. Existem alguns métodos padrão disponíveis, como adicionar uma descrição.

```swift
app.get("hello", ":name") { req in
	...
}.description("says hello")
```

## Grupos de Rotas

O agrupamento de rotas permite criar um conjunto de rotas com um prefixo de caminho ou middleware específico. O agrupamento suporta uma sintaxe baseada em construtor e em closure.

Todos os métodos de agrupamento retornam um `RouteBuilder`, o que significa que você pode misturar, combinar e aninhar seus grupos infinitamente com outros métodos de construção de rotas.

### Prefixo de Caminho

Os grupos de rotas com prefixo de caminho permitem que você adicione um ou mais componentes de caminho a um grupo de rotas.

```swift
let users = app.grouped("users")
// GET /users
users.get { req in
    ...
}
// POST /users
users.post { req in
    ...
}
// GET /users/:id
users.get(":id") { req in
    let id = req.parameters.get("id")!
    ...
}
```

Qualquer componente de caminho que você possa passar para métodos como `get` ou `post` pode ser passado para `grouped`. Existe também uma sintaxe alternativa baseada em closure.

```swift
app.group("users") { users in
    // GET /users
    users.get { req in
        ...
    }
    // POST /users
    users.post { req in
        ...
    }
    // GET /users/:id
    users.get(":id") { req in
        let id = req.parameters.get("id")!
        ...
    }
}
```

Aninhar grupos de rotas com prefixo de caminho permite que você defina APIs CRUD de forma concisa.

```swift
app.group("users") { users in
    // GET /users
    users.get { ... }
    // POST /users
    users.post { ... }

    users.group(":id") { user in
        // GET /users/:id
        user.get { ... }
        // PATCH /users/:id
        user.patch { ... }
        // PUT /users/:id
        user.put { ... }
    }
}
```

### Middleware

Além de adicionar prefixos de caminho, você também pode adicionar middleware a grupos de rotas.

```swift
app.get("fast-thing") { req in
    ...
}
app.group(RateLimitMiddleware(requestsPerMinute: 5)) { rateLimited in
    rateLimited.get("slow-thing") { req in
        ...
    }
}
```

Isso é especialmente útil para proteger subconjuntos das suas rotas com diferentes middlewares de autenticação.

```swift
app.post("login") { ... }
let auth = app.grouped(AuthMiddleware())
auth.get("dashboard") { ... }
auth.get("logout") { ... }
```

## Redirecionamentos

Redirecionamentos são úteis em vários cenários, como encaminhar locais antigos para novos por motivos de SEO, redirecionar um usuário não autenticado para a página de login ou manter a compatibilidade retroativa com a nova versão da sua API.

Para redirecionar uma requisição, use:

```swift
req.redirect(to: "/some/new/path")
```

Você também pode especificar o tipo de redirecionamento. Por exemplo, para redirecionar uma página permanentemente (para que seu SEO seja atualizado corretamente) use:

```swift
req.redirect(to: "/some/new/path", redirectType: .permanent)
```
Os diferentes tipos de `Redirect` são:

* `.permanent` - retorna um redirecionamento **301 Permanent**
* `.normal` - retorna um redirecionamento **303 See Other**. Este é o padrão do Vapor e informa ao cliente para seguir o redirecionamento com uma requisição **GET**.
* `.temporary` - retorna um redirecionamento **307 Temporary**. Isso informa ao cliente para preservar o método HTTP usado na requisição.

> Para escolher o código de status de redirecionamento apropriado, confira [a lista completa](https://pt.wikipedia.org/wiki/Lista_de_c%C3%B3digos_de_status_HTTP#Redirecionamento_3xx)

