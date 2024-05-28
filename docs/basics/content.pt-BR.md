# Conteúdo

A API de conteúdo do Vapor permite que você codifique/decodifique facilmente structs Codable para/de mensagens HTTP. A codificação [JSON](https://tools.ietf.org/html/rfc7159) é usada por padrão com suporte nativo para [Formulário Codificado por URL](https://pt.wikipedia.org/wiki/Codifica%C3%A7%C3%A3o_percentual#O_tipo_application/x-www-form-urlencoded) e [Multipart](https://tools.ietf.org/html/rfc2388). A API também é configurável, permitindo que você adicione, modifique ou substitua estratégias de codificação para determinados tipos de conteúdo HTTP.

## Visão Geral

Para entender como a API de conteúdo do Vapor funciona, você deve primeiro entender alguns conceitos básicos sobre mensagens HTTP. Dê uma olhada no exemplo de requisição a seguir.

```http
POST /greeting HTTP/1.1
content-type: application/json
content-length: 18

{"hello": "world"}
```

Esta requisição indica que contém dados codificados em JSON usando o cabeçalho `content-type` e o tipo de mídia `application/json`. Conforme prometido, alguns dados JSON seguem após os cabeçalhos no corpo.

### Struct de Conteúdo

O primeiro passo para decodificar esta mensagem HTTP é criar um tipo Codable que corresponda à estrutura esperada.

```swift
struct Greeting: Content {
    var hello: String
}
```

Conformar o tipo a `Content` adicionará automaticamente conformidade a `Codable`, juntamente com utilitários adicionais para trabalhar com a API de conteúdo.

Uma vez que você tenha a estrutura de conteúdo, você pode decodificá-la da requisição recebida usando `req.content`.

```swift
app.post("greeting") { req in 
    let greeting = try req.content.decode(Greeting.self)
    print(greeting.hello) // "world"
    return HTTPStatus.ok
}
```

O método decode usa o tipo de conteúdo da requisição para encontrar um decodificador apropriado. Se nenhum decodificador for encontrado, ou se a requisição não contiver o cabeçalho de tipo de conteúdo, um erro `415` será lançado.

Isso significa que essa rota aceita automaticamente todos os outros tipos de conteúdo suportados, como formulário codificado por URL:

```http
POST /greeting HTTP/1.1
content-type: application/x-www-form-urlencoded
content-length: 11

hello=world
```

No caso de uploads de arquivos, sua propriedade de conteúdo deve ser do tipo `Data`.

```swift
struct Profile: Content {
    var name: String
    var email: String
    var image: Data
}
```

### Tipos de Mídia Suportados

Abaixo estão os tipos de mídia que a API de conteúdo suporta por padrão.

| nome               | valor do cabeçalho                 | tipo de mídia              |
|--------------------|------------------------------------|----------------------------|
| JSON               | application/json                   | `.json`                    |
| Multipart          | multipart/form-data                | `.formData`                |
| Formulário Codificado por URL | application/x-www-form-urlencoded | `.urlEncodedForm`          |
| Texto simples      | text/plain                         | `.plainText`               |
| HTML               | text/html                          | `.html`                    |

Nem todos os tipos de mídia suportam todos os recursos de `Codable`. Por exemplo, JSON não suporta fragmentos de nível superior e Texto simples não suporta dados aninhados.

## Consulta

As APIs de Conteúdo do Vapor suportam o tratamento de dados codificados por URL na string de consulta da URL.

### Decodificação

Para entender como a decodificação de uma string de consulta URL funciona, dê uma olhada na requisição de exemplo a seguir.

```http
GET /hello?name=Vapor HTTP/1.1
content-length: 0
```

Assim como as APIs para lidar com o conteúdo do corpo da mensagem HTTP, o primeiro passo para analisar strings de consulta URL é criar uma `struct` que corresponda à estrutura esperada.

```swift
struct Hello: Content {
    var name: String?
}
```

Observe que `name` é uma `String` opcional, já que strings de consulta URL devem sempre ser opcionais. Se você quiser exigir um parâmetro, use um parâmetro de rota em vez disso.

Agora que você tem uma struct `Content` para a string de consulta esperada desta rota, você pode decodificá-la.

```swift
app.get("hello") { req -> String in 
    let hello = try req.query.decode(Hello.self)
    return "Hello, \(hello.name ?? "Anonymous")"
}
```

Esta rota resultaria na seguinte resposta, dada a requisição de exemplo acima:

```http
HTTP/1.1 200 OK
content-length: 12

Hello, Vapor
```

Se a string de consulta fosse omitida, como na seguinte requisição, o nome "Anonymous" seria usado em vez disso.

```http
GET /hello HTTP/1.1
content-length: 0
```

### Valor Único

Além de decodificar para uma struct `Content`, o Vapor também suporta a busca de valores únicos da string de consulta usando subscripts.

```swift
let name: String? = req.query["name"]
```

## Ganchos

O Vapor chamará automaticamente `beforeEncode` e `afterDecode` em um tipo `Content`. Implementações padrão são fornecidas que não fazem nada, mas você pode usar esses métodos para executar lógica personalizada.

```swift
// Executa após este Content ser decodificado. `mutating` é necessário apenas para structs, não para classes.
mutating func afterDecode() throws {
    // O nome pode não ser passado, mas se for, não pode ser uma string vazia.
    self.name = self.name?.trimmingCharacters(in: .whitespacesAndNewlines)
    if let name = self.name, name.isEmpty {
        throw Abort(.badRequest, reason: "Name must not be empty.")
    }
}

// Executa antes deste Content ser codificado. `mutating` é necessário apenas para structs, não para classes.
mutating func beforeEncode() throws {
    // Deve sempre passar um nome de volta, e ele não pode ser uma string vazia.
    guard 
        let name = self.name?.trimmingCharacters(in: .whitespacesAndNewlines), 
        !name.isEmpty 
    else {
        throw Abort(.badRequest, reason: "Name must not be empty.")
    }
    self.name = name
}
```

## Substituir Padrões

Os codificadores e decodificadores padrão usados pelas APIs de Conteúdo do Vapor podem ser configurados.

### Global

`ContentConfiguration.global` permite que você altere os codificadores e decodificadores que o Vapor usa por padrão. Isso é útil para mudar como toda a sua aplicação analisa e serializa dados.

```swift
// cria um novo codificador JSON que usa datas em formato de timestamp Unix
let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .secondsSince1970

// substitui o codificador global usado para o tipo de mídia `.json`
ContentConfiguration.global.use(encoder: encoder, for: .json)
```

Modificar `ContentConfiguration` é geralmente feito em `configure.swift`.

### Uso Único

Chamadas para métodos de codificação e decodificação como `req.content.decode` suportam a passagem de codificadores personalizados para usos únicos.

```swift
// cria um novo decodificador JSON que usa datas em formato de timestamp Unix
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .secondsSince1970

// decodifica a struct Hello usando um decodificador personalizado
let hello = try req.content.decode(Hello.self, using: decoder)
```

## Codificadores Personalizados

Aplicativos e pacotes de terceiros podem adicionar suporte para tipos de mídia que o Vapor não suporta por padrão, criando codificadores personalizados.

### Conteúdo

O Vapor especifica dois protocolos para codificadores capazes de lidar com conteúdo nos corpos de mensagens HTTP: `ContentDecoder` e `ContentEncoder`.

```swift
public protocol ContentEncoder {
    func encode<E>(_ encodable: E, to body: inout ByteBuffer, headers: inout HTTPHeaders) throws
        where E: Encodable
}

public protocol ContentDecoder {
    func decode<D>(_ decodable: D.Type, from body: ByteBuffer, headers: HTTPHeaders) throws -> D
        where D: Decodable
}
```

Conformar-se a esses protocolos permite que seus codificadores personalizados sejam registrados em `ContentConfiguration`, conforme especificado acima.

### Consulta de URL

O Vapor especifica dois protocolos para codificadores capazes de lidar com conteúdo em strings de consulta de URL: `URLQueryDecoder` e `URLQueryEncoder`.

```swift
public protocol URLQueryDecoder {
    func decode<D>(_ decodable: D.Type, from url: URI) throws -> D
        where D: Decodable
}

public protocol URLQueryEncoder {
    func encode<E>(_ encodable: E, to url: inout URI) throws
        where E: Encodable
}
```

Conformar-se a esses protocolos permite que seus codificadores personalizados sejam registrados em `ContentConfiguration` para lidar com strings de consulta de URL usando os métodos `use(urlEncoder:)` e `use(urlDecoder:)`.

### `ResponseEncodable` Personalizado

Outra abordagem envolve a implementação de `ResponseEncodable` em seus tipos. Considere este tipo de embrulho `HTML` trivial:

```swift
struct HTML {
  let value: String
}
```

Em seguida, sua implementação `ResponseEncodable` seria assim:

```swift
extension HTML: ResponseEncodable {
  public func encodeResponse(for request: Request) -> EventLoopFuture<Response> {
    var headers = HTTPHeaders()
    headers.add(name: .contentType, value: "text/html")
    return request.eventLoop.makeSucceededFuture(.init(
      status: .ok, headers: headers, body: .init(string: value)
    ))
  }
}
```

Se estiver usando `async`/`await`, você pode usar `AsyncResponseEncodable`:

```swift
extension HTML: AsyncResponseEncodable {
  public func encodeResponse(for request: Request) async throws -> Response {
    var headers = HTTPHeaders()
    headers.add(name: .contentType, value: "text/html")
    return .init(status: .ok, headers: headers, body: .init(string: value))
  }
}
```

Observe que isso permite personalizar o cabeçalho `Content-Type`. Consulte a [referência do HTTPHeaders](https://api.vapor.codes/vapor/documentation/vapor/response/headers) para mais detalhes.

Em seguida, você pode usar `HTML` como um tipo de resposta em suas rotas:

```swift
app.get { _ in
  HTML(value: """
  <html>
    <body>
      <h1>Hello, World!</h1>
    </body>
  </html>
  """)
}
```
