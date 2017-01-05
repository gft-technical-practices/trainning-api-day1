# PráticaTecnológica@GFT: 

# Treinamento - API - Dia #1: Conceitos e Modelagem

## Requisitos:

- Powershell
- Chocolatey
- Git
- NodeJs
- Npm
- MongoDb
- Postman
- Atom
- Swagger Codegen

## Configurando seu Ambiente de Desenvolvimento:

- Criar arquivo setup-environment.bat com script abaixo:

***
```bat
REM Resolve PowerShell
IF EXIST c:\windows\sysnative\NUL (
	SET ps="c:\windows\sysnative\windowspowershell\v1.0\powershell.exe"
) else (
	SET ps="c:\windows\system32\windowspowershell\v1.0\powershell.exe"
)
REM Install PowerShell 4
WHERE choco
IF %ERRORLEVEL% NEQ 0 %PS% -NoProfile -ExecutionPolicy unrestricted -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%systemdrive%\ProgramData\chocolatey\bin
choco install powershell4 --acceptlicense -y
REM Install Requirements
choco install git --acceptlicense -y
choco install nodejs --acceptlicense -y
choco install npm --acceptlicense -y
choco install atom --acceptlicense -y
choco install postman --acceptlicense -y
***
```

## Instalando sua Aplicação:

- Clonar o repositório: `git clone git@github.com:gft-technical-practices/trainning-api-day1`
- Instalar dependencias: `npm install`
- Iniciar aplicação: `node server.js`

## Editor para criar suas APIs:

- [Atom](https://atom.io)
- [Extensão: Api-Workbench](https://atom.io/packages/api-workbench)

## Testando sua API:

- [Postman Installer](https://www.getpostman.com)
- [Postman Chrome Plugin](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)

***
## Exercício de Modelagem:

## Mas antes de tudo, o que exatamente é o Swagger?

O Swagger é um projeto composto por algumas ferramentas que auxiliam o desenvolvedor de APIs REST em algumas tarefas como:

- Modelagem da API
- Geração de documentação (legível) da API
- Geração de códigos do Cliente e do Servidor, com suporte a várias linguagens de programação

Para isso, o Swagger especifica a OpenAPI, uma linguagem para descrição de contratos de APIs REST. A OpenAPI define um formato JSON com campos padronizados (através de um JSON Schema) para que você descreva recursos, modelo de dados, URIs, Content-Types, métodos HTTP aceitos e códigos de resposta. Também pode ser utilizado o formato YAML.

Além da OpenAPI, o Swagger provê um ecossistema de ferramentas. 

As principais são:

- [Swagger Editor](http://swagger.io/swagger-editor): para criação de contratos
- [Swagger UI](http://swagger.io/swagger-ui): para publicação da documentação
- [Swagger Codegen](http://swagger.io/swagger-codegen): para geração de “esqueletos” de servidores em mais de 10 tecnologias e de clientes em mais de 25 tecnologias diferentes

***

## Modelando a API da PagueRapido

Vamos modelar a API da PagueRapido, uma aplicação de pagamentos bem simples que é estudada no curso SOA na prática.

Pra começar, devemos definir algumas informações iniciais, como a versão do Swagger que estamos usando:
```
swagger: '2.0'
```
O título, descrição e versão da API devem ser definidos:
```
info:
  title: PagueRapido API
  description: Pagamentos rápidos
  version: 1.0.0
```
Em host, inserimos o endereço do servidor da API, em basePath colocamos o contexto da aplicação e em schemes informamos se a aplicação aceita HTTP e/ou HTTPS.
```
host: localhost:8080
basePath: /fj36-PagueRapido/v1
schemes:
  - http
  - https
```

## Defininindo o modelo de dados

De alguma forma, precisamos definir quais dados são recebidos e retornados pela API.

Na nossa API, recebemos dados de uma Transação, que tem um código, titular, data e valor. A partir disso, geramos um Pagamento com id, status e valor.

### Modelo de dados da API do PagueRapido:

### Transacao 	

| Coluna		| Tipo de Dado          
| ------------- |:-------------:
| numero		| string
| titular	    | string
| valor			| date
| valor			| decimal


### Pagamento

| Coluna		| Tipo de Dado          
| ------------- |:-------------:
| id			| integer
| status		| string
| valor			| decimal





Em um WSDL, esse modelo de dados é definido através de um XML Schema (XSD). No caso do Swagger, o modelo de dados fica em um JSON Schema na seção definitions do contrato.

De acordo com o JSON Schema, em type podemos usar tipos primitivos de dados para números inteiros (integer), números decimais (number), textos (string) e booleanos (boolean). Esses tipos primitivos podem ser modificados com a propriedade format. 

Para o tipo integer temos os formatos int32 (32 bits) e int64 (64 bits, ou long). 

Para o number, temos os formatos float e double. Não há um tipo específico para datas, então temos que utilizar uma string com o formato date (só data) ou date-time (data e hora).

Além dos tipos primitivos, podemos definir objetos com um type igual a object. Esses objetos são compostos por várias outras propriedades, que ficam em properties.

No nosso caso, o modelo de dados com os objetos Transacao e Pagamento ficaria algo como:

```
definitions:
  Transacao:
    type: object
    properties:
      codigo:
        type: string
      titular:
        type: string
      data:
        type: string
        format: date
      valor:
        type: number
        format: double
  Pagamento:
    type: object
    properties:
      id:
        type: integer
        format: int32
      status:
        type: string
      valor:
        type: number
        format: double

```

## Defininindo os recursos da API

Com o modelo de dados pronto, precisamos modelar os recursos da nossa API e as respectivas URIs. No PagueRapido, teremos o recurso Pagamento acessível pela URI /pagamentos.

Um POST em /pagamentos cria um novo pagamento. Se o pagamento criado tiver o id 1, por exemplo, as informações estariam acessíveis na URI /pagamentos/1.

Podemos fazer duas coisas com o nosso pagamento: para confirmá-lo, devemos enviar um PUT para /pagamentos/1; para cancelá-lo, enviamos um DELETE.

No Swagger, as URIs devem ser descritas na seção paths:
```
paths:
  /pagamentos:
    post:
      summary: Cria novo pagamento
 
  /pagamentos/{id}:
    put:
      summary: Confirma um pagamento
    delete:
      summary: Cancela um pagamento

```

## Definindo os parâmetros de request

Para a URI /pagamentos, que recebe um POST, é enviada uma transação no corpo da requisição, que deve estar no formato JSON. É feita uma referência ao modelo Transacao definido anteriormente.
```
paths:
  /pagamentos:
    post:
      summary: Cria novo pagamento
      consumes:
        - application/json
      parameters:
        - in: body
          name: transacao
          required: true
          schema:
            $ref: '#/definitions/Transacao'
```
Já para a URI /pagamentos/{id}, é definido um path parameter com o id do pagamento. Esse parâmetro pode ser descrito na seção parameters, logo acima da seção paths, e depois referenciado nos métodos.
```
parameters:
  pagamento-id:
    name: id
    in: path
    description: id do pagamento
    type: integer
    format: int32
    required: true
paths:
  /pagamentos:
    #código omitido... 
 
  /pagamentos/{id}:
    put:
      summary: Confirma um pagamento
      parameters:
        - $ref: '#/parameters/pagamento-id'
    delete:
      summary: Cancela um pagamento
      parameters:
        - $ref: '#/parameters/pagamento-id'
```

## Definindo os tipos de response

Definidos os parâmetros de request, precisamos modelar o response.

Depois da criação do pagamento, deve ser retornado um response com o status 201 (Created) juntamente com a URI do novo pagamento no header Location e uma representação em JSON no corpo da resposta.
```
paths:
  /pagamentos:
    post:
      summary: Cria novo pagamento
      consumes:
        - application/json
      produces:
        - application/json
      #código omitido...
      responses:
        '201':
          description: Novo pagamento criado
          schema:
            $ref: '#/definitions/Pagamento'
          headers:
            Location:
              description: uri do novo pagamento
              type: string
```

Após a confirmação de um pagamento, é simplesmente retornado o status 200 (OK). O mesmo retorno acontece após um cancelamento.
```
/pagamentos/{id}:
    put:
      summary: Confirma um pagamento
      parameters:
        - $ref: '#/parameters/pagamento-id'
      responses:
        '200':
          description: 'Pagamento confirmado'
    delete:
      summary: Cancela um pagamento
      parameters:
        - $ref: '#/parameters/pagamento-id'
      responses:
        '200':
          description: 'Pagamento cancelado'
```

## Concluindo

A abordagem utilizada nesse post foi a conhecida como Contract-First ou API-First Development.

Modelamos a API pensando nos dados, nos recursos, URIs, métodos, parâmetros e respostas. Começamos a definição do serviço criando primeiro a API de comunicação, para só posteriormente pensar na implementação.

Esta abordagem gera um desacoplamento entre implementação e interface de uso, além de permitir que tando o lado cliente quanto o lado servidor possam iniciar seu desenvolvimento assim que a API estiver definida, mesmo sem uma implementação finalizada.
