# Aula Prática 10.1 – Primeiros Testes de Integração com Postman

## Objetivos

Nesta aula prática, utilizaremos o projeto **Menu-Back-End** e a ferramenta **Postman** para compreender, de forma simples, como realizar testes de integração em uma API REST.

Ao final da atividade, você deverá ser capaz de:

- executar a aplicação `Menu-Back-End`;
- compreender os principais elementos do Postman;
- enviar requisições HTTP;
- organizar requisições em uma Collection;
- utilizar variáveis de ambiente;
- compreender e criar scripts de pré-requisição e pós-resposta;
- criar verificações automatizadas;
- executar um fluxo CRUD completo;
- executar uma Collection como uma suíte automatizada;
- compreender a ideia de integração incremental;
- relacionar integração incremental e regressão.

---

# 1. Sistema utilizado

Nesta aula será utilizado o projeto:

**Menu-Back-End**

```text
https://github.com/Herysson/Menu-Back-End
```

O projeto disponibiliza uma API REST desenvolvida com:

- Java;
- Spring Boot;
- Spring Data JPA;
- MySQL;
- Maven.

A API permite realizar operações CRUD sobre produtos.

## Endpoints utilizados

| Método | Endpoint | Operação |
|---|---|---|
| `GET` | `/produtos` | Listar todos os produtos |
| `GET` | `/produtos/{id}` | Buscar um produto por identificador |
| `POST` | `/produtos` | Cadastrar um produto |
| `PUT` | `/produtos/{id}` | Atualizar um produto |
| `DELETE` | `/produtos/{id}` | Excluir um produto |

---

# 2. O que é teste de integração?

Em um teste unitário, normalmente verificamos uma unidade isolada.

Exemplo:

```text
ProdutoService
```

Em um teste de integração, queremos verificar se diferentes partes do sistema funcionam corretamente **quando utilizadas em conjunto**.

Ao enviar uma requisição pelo Postman, o fluxo pode envolver:

```text
Postman
   ↓
HTTP
   ↓
Spring Boot
   ↓
Controller
   ↓
JPA / Repository
   ↓
MySQL
```

O resultado percorre o caminho inverso até ser transformado em uma resposta HTTP.

Portanto, uma falha pode estar:

- na requisição;
- no formato do JSON;
- no Controller;
- no mapeamento JPA;
- na persistência;
- na comunicação com o banco;
- na resposta produzida pela aplicação.

> Uma unidade funcionar corretamente de forma isolada não garante que funcionará corretamente quando integrada às demais.

---

# 3. Pré-requisitos

Antes de iniciar, certifique-se de possuir:

- Java 17 ou superior;
- MySQL;
- Git;
- uma IDE Java;
- Postman;
- acesso ao projeto `Menu-Back-End`.

---

# 4. Preparando o projeto

Clone o repositório:

```bash
git clone https://github.com/Herysson/Menu-Back-End.git
```

Entre na pasta:

```bash
cd Menu-Back-End
```

Configure:

```text
src/main/resources/application.properties
```

Exemplo:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/menu?createDatabaseIfNotExist=true
spring.datasource.username=SEU_USUARIO
spring.datasource.password=SUA_SENHA

spring.jpa.hibernate.ddl-auto=update
spring.jpa.open-in-view=true
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

Execute a aplicação:

```bash
./mvnw spring-boot:run
```

No Windows:

```cmd
mvnw.cmd spring-boot:run
```

A API deverá ficar disponível em:

```text
http://localhost:8080
```

---

# 5. Conhecendo a terminologia do Postman

Antes de começar, é importante utilizar a nomenclatura correta da ferramenta.

| Termo | Significado |
|---|---|
| **Request** | Requisição enviada à API |
| **Response** | Resposta recebida da API |
| **Collection** | Agrupamento de requisições relacionadas |
| **Environment** | Conjunto de variáveis associado a um ambiente |
| **Variable** | Valor reutilizável, como `baseUrl` ou `produtoId` |
| **Pre-request script** | Script executado antes do envio da Request |
| **Post-response script** | Script executado depois que a Response é recebida |
| **Test** | Verificação definida normalmente com `pm.test()` |
| **Collection Runner** | Recurso utilizado para executar várias Requests e seus Tests em sequência |

Uma Request pode existir sem possuir Tests.

Exemplo:

```text
GET /produtos
```

Isso é uma **Request**.

Já:

```javascript
pm.test("A resposta deve indicar sucesso", function () {
    pm.response.to.be.success;
});
```

é um **Test** executado em um **Post-response script**.

## Documentação

Postman – Collections:

```text
https://learning.postman.com/docs/collections/collections-overview/
```

Postman – Scripts:

```text
https://learning.postman.com/docs/tests-and-scripts/write-scripts/intro-to-scripts/
```

---

# 6. Primeiro GET

Abra o Postman e crie uma nova Request:

```http
GET http://localhost:8080/produtos
```

Execute utilizando:

```text
Send
```

Observe:

- status HTTP;
- corpo da resposta;
- formato JSON;
- tempo de resposta.

Se a aplicação estiver funcionando corretamente, deverá ser retornada a lista de produtos.

## Reflexão

Se esse teste falhar, a causa obrigatoriamente está no Controller?

**Não.**

A falha pode estar em qualquer ponto:

```text
Postman
   ↓
Spring Boot
   ↓
JPA
   ↓
MySQL
```

---

# 7. Criando uma Collection

No Postman:

1. acesse **Collections**;
2. clique em **Create Collection**;
3. crie:

```text
Menu Back-End - Testes de Integração
```

Dentro dela, crie:

```text
01 - Fluxo CRUD
```

Ao final da prática teremos:

```text
Menu Back-End - Testes de Integração
│
└── 01 - Fluxo CRUD
    ├── GET - Listar produtos
    ├── POST - Criar produto
    ├── GET - Localizar produto
    ├── GET - Buscar produto por ID
    ├── PUT - Atualizar produto
    ├── GET - Confirmar atualização
    ├── DELETE - Excluir produto
    └── GET - Confirmar exclusão
```

Salve a primeira Request como:

```text
GET - Listar produtos
```

## Documentação

```text
https://learning.postman.com/docs/collections/collections-overview/
```

---

# 8. Criando um Environment

Crie um Environment:

```text
Menu - Local
```

Adicione:

| Variable | Value |
|---|---|
| `baseUrl` | `http://localhost:8080` |

Selecione o Environment no Postman.

Agora substitua:

```text
http://localhost:8080/produtos
```

por:

```text
{{baseUrl}}/produtos
```

Quando a Request for executada, o Postman substituirá:

```text
{{baseUrl}}
```

por:

```text
http://localhost:8080
```

Portanto:

```text
{{baseUrl}}/produtos
```

torna-se:

```text
http://localhost:8080/produtos
```

## Por que utilizar variáveis?

Imagine dois ambientes:

```text
LOCAL
http://localhost:8080
```

```text
TESTE
https://teste.exemplo.com
```

Não precisamos alterar todas as Requests. Basta modificar ou selecionar outro Environment.

## Documentação

```text
https://learning.postman.com/docs/sending-requests/variables/managing-environments/
```

---

# 9. Scripts no Postman

O Postman permite executar JavaScript durante o ciclo de uma Request.

Nesta aula utilizaremos dois momentos:

```text
Pre-request
    ↓
REQUEST
    ↓
RESPONSE
    ↓
Post-response
```

## Pre-request script

É executado **antes** do envio da Request.

Pode ser usado para:

- criar dados;
- definir variáveis;
- preparar valores dinâmicos.

## Post-response script

É executado **depois** que a Response é recebida.

Pode ser usado para:

- verificar resultados;
- ler o JSON retornado;
- armazenar dados da resposta;
- criar Tests.

> Nesta aula utilizaremos scripts no nível da própria Request para facilitar a compreensão.

## Documentação

```text
https://learning.postman.com/docs/tests-and-scripts/write-scripts/intro-to-scripts/
```

---

# 10. Criando o primeiro Test

Na Request:

```text
GET - Listar produtos
```

abra:

```text
Scripts → Post-response
```

Adicione:

```javascript
pm.test("A resposta deve indicar sucesso", function () {
    pm.response.to.be.success;
});
```

Vamos entender esse código.

## `pm`

```javascript
pm
```

é o objeto disponibilizado pelo Postman para acessar funcionalidades da ferramenta.

## `pm.test()`

```javascript
pm.test(...)
```

cria um Test.

## Nome do Test

```javascript
"A resposta deve indicar sucesso"
```

é o nome que aparecerá no resultado da execução.

## `pm.response`

```javascript
pm.response
```

representa a Response recebida.

## Verificação

```javascript
pm.response.to.be.success;
```

verifica se a resposta pertence à faixa de sucesso HTTP.

Execute novamente. O resultado será apresentado como:

```text
PASS
```

ou:

```text
FAIL
```

## Documentação

```text
https://learning.postman.com/docs/tests-and-scripts/write-scripts/test-scripts/
```

---

# 11. Criando mais verificações

Adicione:

```javascript
pm.test("A resposta deve ser JSON", function () {
    pm.response.to.be.json;
});
```

Depois:

```javascript
pm.test("O corpo da resposta deve ser uma lista", function () {
    const body = pm.response.json();
    pm.expect(body).to.be.an("array");
});
```

## Entendendo `pm.response.json()`

```javascript
const body = pm.response.json();
```

converte o JSON recebido em um objeto JavaScript.

Exemplo de Response:

```json
[
  {
    "id": 1,
    "nome": "X-Bacon"
  }
]
```

## Entendendo `pm.expect()`

```javascript
pm.expect(body).to.be.an("array");
```

significa:

> Esperamos que `body` seja um array.

Assim:

```text
pm.test()
```

define o Test, enquanto:

```text
pm.expect()
```

define uma expectativa dentro do Test.

---

# 12. Executar não é o mesmo que testar

Enviar uma Request e observar a Response é útil.

Mas um teste precisa possuir um **resultado esperado**.

```text
Entrada
   ↓
Execução
   ↓
Resultado obtido
   ↓
Comparação
   ↓
Resultado esperado
```

Sem uma expectativa, temos apenas execução.

Quando escrevemos:

```javascript
pm.expect(body).to.be.an("array");
```

estamos transformando uma observação em uma verificação automatizada.

---

# 13. Criando um produto

Crie:

```text
POST - Criar produto
```

Request:

```http
POST {{baseUrl}}/produtos
```

Selecione:

```text
Body → raw → JSON
```

Utilize:

```json
{
  "nome": "Clássico Bacon",
  "descricao": "Pão brioche, hambúrguer 180g, queijo cheddar e bacon.",
  "preco": 29.90,
  "categoria": "Hambúrguer",
  "disponibilidade": true,
  "imagem": "https://exemplo.com/bacon.jpg"
}
```

Ao utilizar:

```text
raw → JSON
```

o Postman deve enviar o cabeçalho:

```text
Content-Type: application/json
```

Sem o corpo JSON, o Spring poderá responder:

```text
400 Bad Request
```

com uma mensagem semelhante a:

```text
Required request body is missing
```

---

# 14. Criando um nome dinâmico com Pre-request script

Se executarmos o teste várias vezes utilizando sempre o mesmo nome, poderemos criar muitos registros iguais.

Para identificar facilmente os dados criados pelo teste, abra:

```text
Scripts → Pre-request
```

e adicione:

```javascript
pm.environment.set(
    "nomeProdutoTeste",
    "Produto Teste " + Date.now()
);
```

## O que esse script faz?

```javascript
Date.now()
```

gera um valor numérico baseado no instante atual.

Por exemplo:

```text
1794255827365
```

Então:

```javascript
"Produto Teste " + Date.now()
```

pode produzir:

```text
Produto Teste 1794255827365
```

Depois:

```javascript
pm.environment.set(...)
```

armazena esse valor no Environment ativo.

Teremos:

| Variable | Value |
|---|---|
| `baseUrl` | `http://localhost:8080` |
| `nomeProdutoTeste` | `Produto Teste 1794255827365` |

Agora altere o Body:

```json
{
  "nome": "{{nomeProdutoTeste}}",
  "descricao": "Produto criado durante o teste de integração.",
  "preco": 29.90,
  "categoria": "Hambúrguer",
  "disponibilidade": true,
  "imagem": "https://exemplo.com/produto-teste.jpg"
}
```

---

# 15. Verificando a criação

Em:

```text
Scripts → Post-response
```

adicione:

```javascript
pm.test("A criação deve retornar sucesso", function () {
    pm.response.to.be.success;
});
```

Se a Response retornar o produto criado:

```javascript
pm.test("O nome retornado deve ser o nome enviado", function () {
    const body = pm.response.json();

    pm.expect(body.nome)
        .to.eql(pm.environment.get("nomeProdutoTeste"));
});
```

## `pm.environment.get()`

```javascript
pm.environment.get("nomeProdutoTeste")
```

recupera o valor previamente armazenado no Environment.

Assim podemos comparar:

```text
nome enviado
        ×
nome retornado
```

---

# 16. Armazenando o ID do produto

As próximas Requests precisam saber **qual produto foi criado**.

Suponha que a Response seja:

```json
{
  "id": 7,
  "nome": "Produto Teste 1794255827365",
  "preco": 29.90
}
```

No Post-response, adicione:

```javascript
const body = pm.response.json();

if (body.id !== undefined && body.id !== null) {
    pm.environment.set("produtoId", body.id);
}
```

## Passo a passo

Primeiro:

```javascript
const body = pm.response.json();
```

lê o JSON.

Depois:

```javascript
body.id
```

retorna:

```text
7
```

Finalmente:

```javascript
pm.environment.set("produtoId", body.id);
```

armazena:

| Variable | Value |
|---|---|
| `produtoId` | `7` |

A variável fica armazenada no Environment:

```text
Menu - Local
```

Agora podemos utilizar:

```text
{{produtoId}}
```

em outras Requests.

## Documentação

```text
https://learning.postman.com/docs/sending-requests/variables/managing-environments/
```

---

# 17. E se o POST não retornar o ID?

Se a implementação não devolver o objeto criado, não devemos inventar o identificador.

Podemos localizá-lo pela listagem.

Crie:

```text
GET - Localizar produto
```

Request:

```http
GET {{baseUrl}}/produtos
```

Post-response:

```javascript
const produtos = pm.response.json();
const nome = pm.environment.get("nomeProdutoTeste");

const produto = produtos.find(
    p => p.nome === nome
);

pm.test("O produto criado deve existir", function () {
    pm.expect(produto).to.not.be.undefined;
});

if (produto) {
    pm.environment.set("produtoId", produto.id);
}
```

O fluxo é:

```text
GET /produtos
      ↓
recebe lista
      ↓
procura nomeProdutoTeste
      ↓
encontra Produto
      ↓
lê produto.id
      ↓
salva produtoId
```

---

# 18. Buscando o produto pelo ID

Crie:

```text
GET - Buscar produto por ID
```

Request:

```http
GET {{baseUrl}}/produtos/{{produtoId}}
```

Imagine:

```text
baseUrl = http://localhost:8080
produtoId = 7
```

Então:

```text
{{baseUrl}}/produtos/{{produtoId}}
```

torna-se:

```text
http://localhost:8080/produtos/7
```

Adicione:

```javascript
pm.test("A consulta deve retornar sucesso", function () {
    pm.response.to.be.success;
});
```

Depois:

```javascript
pm.test("O produto retornado deve possuir o ID esperado", function () {
    const body = pm.response.json();

    pm.expect(String(body.id))
        .to.eql(String(pm.environment.get("produtoId")));
});
```

E:

```javascript
pm.test("O produto deve manter o nome cadastrado", function () {
    const body = pm.response.json();

    pm.expect(body.nome)
        .to.eql(pm.environment.get("nomeProdutoTeste"));
});
```

---

# 19. O que foi integrado até aqui?

Até aqui temos:

```text
POST
 ↓
Spring Boot
 ↓
persistência
 ↓
MySQL
```

e depois:

```text
GET
 ↓
Spring Boot
 ↓
MySQL
 ↓
JSON
```

Um dado criado por uma operação foi recuperado por outra.

Isso verifica a integração entre:

- HTTP;
- Controller;
- persistência;
- banco de dados;
- conversão para JSON.

---

# 20. Atualizando o produto

Crie:

```text
PUT - Atualizar produto
```

Request:

```http
PUT {{baseUrl}}/produtos/{{produtoId}}
```

Body:

```json
{
  "nome": "{{nomeProdutoTeste}}",
  "descricao": "Produto atualizado durante o teste de integração.",
  "preco": 35.90,
  "categoria": "Hambúrguer",
  "disponibilidade": false,
  "imagem": "https://exemplo.com/produto-teste.jpg"
}
```

Post-response:

```javascript
pm.test("A atualização deve retornar sucesso", function () {
    pm.response.to.be.success;
});
```

---

# 21. Confirmando a persistência

Uma resposta de sucesso do `PUT` não garante, sozinha, que o dado foi persistido corretamente.

Crie:

```text
GET - Confirmar atualização
```

Request:

```http
GET {{baseUrl}}/produtos/{{produtoId}}
```

Adicione:

```javascript
pm.test("O preço atualizado deve ter sido persistido", function () {
    const body = pm.response.json();

    pm.expect(Number(body.preco)).to.eql(35.90);
});
```

Depois:

```javascript
pm.test("O produto deve estar indisponível", function () {
    const body = pm.response.json();

    pm.expect(body.disponibilidade).to.eql(false);
});
```

Temos:

```text
PUT
 ↓
persistência
 ↓
GET
 ↓
confirmação
```

---

# 22. Excluindo o produto

Crie:

```text
DELETE - Excluir produto
```

Request:

```http
DELETE {{baseUrl}}/produtos/{{produtoId}}
```

Post-response:

```javascript
pm.test("A exclusão deve retornar sucesso", function () {
    pm.response.to.be.success;
});
```

---

# 23. Confirmando a exclusão

Crie:

```text
GET - Confirmar exclusão
```

Request:

```http
GET {{baseUrl}}/produtos
```

Post-response:

```javascript
const produtos = pm.response.json();
const id = String(pm.environment.get("produtoId"));

const encontrado = produtos.some(
    p => String(p.id) === id
);

pm.test("O produto excluído não deve aparecer na listagem", function () {
    pm.expect(encontrado).to.eql(false);
});
```

Não estamos verificando apenas se:

```text
DELETE → sucesso
```

Também confirmamos:

```text
DELETE
 ↓
GET
 ↓
registro não encontrado
```

---

# 24. O fluxo completo

Ao final:

```text
POST
 ↓
criar produto
 ↓
GET
 ↓
confirmar criação
 ↓
PUT
 ↓
GET
 ↓
confirmar atualização
 ↓
DELETE
 ↓
GET
 ↓
confirmar exclusão
```

O objetivo não é apenas testar endpoints isoladamente.

Também queremos verificar se diferentes operações conseguem trabalhar de forma consistente sobre os mesmos dados.

---

# 25. Executando todos os testes com o Collection Runner

Até agora executamos as Requests individualmente utilizando:

```text
Send
```

Mas o objetivo de uma suíte automatizada é poder executar várias Requests e Tests em sequência.

No Postman, utilize o recurso:

```text
Collection Runner
```

Selecione:

```text
Menu Back-End - Testes de Integração
```

ou a pasta:

```text
01 - Fluxo CRUD
```

Selecione também:

```text
Environment: Menu - Local
```

Confira a ordem:

```text
GET - Listar produtos
POST - Criar produto
GET - Localizar produto
GET - Buscar produto por ID
PUT - Atualizar produto
GET - Confirmar atualização
DELETE - Excluir produto
GET - Confirmar exclusão
```

Execute a Collection.

## Documentação

```text
https://learning.postman.com/docs/collections/running-collections/intro-to-collection-runs/
```

---

# 26. O que o Collection Runner executa?

O Runner executa:

```text
Request 1
   ↓
Tests da Request 1
   ↓
Request 2
   ↓
Tests da Request 2
   ↓
Request 3
   ↓
Tests da Request 3
   ↓
...
```

Um resultado pode ser semelhante a:

```text
GET - Listar produtos
✓ resposta indica sucesso
✓ resposta é JSON
✓ corpo é lista

POST - Criar produto
✓ criação indica sucesso
✓ nome retornado está correto

GET - Buscar produto por ID
✓ consulta indica sucesso
✓ ID está correto

PUT - Atualizar produto
✓ atualização indica sucesso

GET - Confirmar atualização
✓ preço atualizado
✓ disponibilidade atualizada

DELETE - Excluir produto
✓ exclusão indica sucesso

GET - Confirmar exclusão
✓ produto não aparece mais
```

---

# 27. O que significa automatizar?

Sem automação:

```text
Send GET
 ↓
verificar manualmente

Send POST
 ↓
verificar manualmente

Send PUT
 ↓
verificar manualmente
```

Com automação:

```text
Collection Runner
       ↓
GET
       ↓
Tests
       ↓
POST
       ↓
Tests
       ↓
PUT
       ↓
Tests
       ↓
DELETE
       ↓
Tests
       ↓
resultado da execução
```

Automatizar não significa apenas enviar Requests automaticamente.

Também significa verificar automaticamente se os resultados obtidos correspondem ao esperado.

---

# 28. Integração incremental

Uma estratégia importante é integrar e testar gradualmente.

Em vez de:

```text
integrar tudo
     ↓
testar tudo no final
```

podemos trabalhar:

```text
integrar
   ↓
testar
   ↓
adicionar novo elemento
   ↓
testar novamente
```

Nesta prática utilizaremos essa ideia de forma didática.

---

# 29. Incremento 1 – Listagem

Primeiro temos apenas:

```text
GET /produtos
```

Executamos:

```text
GET
 ↓
Tests
```

Verificamos:

- aplicação acessível;
- banco acessível;
- JSON retornado;
- estrutura básica correta.

---

# 30. Incremento 2 – Criação

Adicionamos:

```text
POST /produtos
```

Agora executamos:

```text
POST
 ↓
GET
```

Pergunta:

> O produto criado pode ser recuperado?

Também reexecutamos o teste anterior.

---

# 31. Incremento 3 – Busca por ID

Adicionamos:

```text
GET /produtos/{id}
```

Fluxo:

```text
POST
 ↓
produtoId
 ↓
GET /{id}
```

Mantemos os testes já existentes.

---

# 32. Incremento 4 – Atualização

Adicionamos:

```text
PUT /produtos/{id}
```

Agora:

```text
POST
 ↓
GET
 ↓
PUT
 ↓
GET
```

A nova consulta confirma a persistência.

---

# 33. Incremento 5 – Exclusão

Adicionamos:

```text
DELETE /produtos/{id}
```

Fluxo completo:

```text
POST
 ↓
GET
 ↓
PUT
 ↓
GET
 ↓
DELETE
 ↓
GET
```

Depois executamos toda a Collection.

---

# 34. Por que reexecutar os testes anteriores?

Considere:

```text
GET → PASS
POST → PASS
GET ID → PASS
```

Adicionamos:

```text
PUT
```

Depois executamos novamente:

```text
GET → PASS
POST → PASS
GET ID → FAIL
PUT → PASS
```

Mesmo que o teste novo tenha passado, um comportamento anterior deixou de funcionar.

Isso caracteriza uma possível **regressão**.

---

# 35. Introdução ao teste de regressão

Teste de regressão consiste em reexecutar testes anteriormente executados para verificar se uma alteração ou nova integração introduziu efeitos indesejados.

Podemos representar:

```text
Integração nova
      ↓
novos Tests
      +
Tests anteriores
      ↓
reexecução
```

No nosso exemplo:

```text
Incremento 1
GET

Incremento 2
GET + POST

Incremento 3
GET + POST + GET ID

Incremento 4
GET + POST + GET ID + PUT + confirmação

Incremento 5
fluxo completo
```

A Collection facilita essa reexecução.

---

# 36. Integração incremental + automação + regressão

Os três conceitos se relacionam:

```text
INTEGRAÇÃO INCREMENTAL
        ↓
novos componentes/comportamentos
        ↓
AUTOMAÇÃO
        ↓
executar a suíte novamente
        ↓
REGRESSÃO
        ↓
verificar se algo anteriormente correto deixou de funcionar
```

Essa é uma das principais razões para automatizar testes de integração.

---

# 37. Atividade prática

Utilizando o projeto `Menu-Back-End`, construa no Postman uma Collection que:

1. liste os produtos;
2. cadastre um novo produto;
3. armazene o identificador do produto;
4. consulte o produto pelo identificador;
5. altere pelo menos dois campos;
6. confirme a persistência da alteração;
7. exclua o produto;
8. confirme a exclusão.

Cada Request deverá possuir ao menos um Test quando aplicável.

---

# 38. Execução automatizada

Depois de concluir a Collection:

1. abra o Collection Runner;
2. selecione o Environment `Menu - Local`;
3. confira a ordem das Requests;
4. execute toda a Collection;
5. observe os Tests aprovados e reprovados;
6. corrija eventuais problemas;
7. execute novamente.

Registre o resultado final da execução.

---

# 39. Questões para discussão

1. Qual a diferença entre uma Request e um Test no Postman?
2. Para que serve um Pre-request script?
3. Para que serve um Post-response script?
4. Onde a variável `produtoId` é armazenada?
5. Por que o `produtoId` é importante para o fluxo da Collection?
6. Qual a função de `pm.test()`?
7. Qual a função de `pm.expect()`?
8. Por que confirmamos a atualização utilizando um novo GET?
9. O que o Collection Runner automatiza?
10. Qual a diferença entre executar Requests individualmente e executar a Collection?
11. O que significa integração incremental?
12. Por que reexecutar testes anteriores?
13. O que é uma regressão?
14. Se todos os Tests passarem, podemos afirmar que o sistema não possui defeitos?

---

# 40. Checklist

Antes de finalizar:

- [ ] A aplicação Spring Boot está executando.
- [ ] O MySQL está configurado.
- [ ] A Collection foi criada.
- [ ] O Environment `Menu - Local` está selecionado.
- [ ] `baseUrl` está configurada.
- [ ] O primeiro GET funciona.
- [ ] Existem Tests no Post-response.
- [ ] O POST envia Body em JSON.
- [ ] O Pre-request cria `nomeProdutoTeste`.
- [ ] O Post-response salva `produtoId`.
- [ ] O GET por ID utiliza `{{produtoId}}`.
- [ ] O PUT foi executado.
- [ ] A alteração foi confirmada por GET.
- [ ] O DELETE foi executado.
- [ ] A exclusão foi confirmada.
- [ ] A Collection foi executada no Runner.
- [ ] Todos os Tests foram analisados.
- [ ] Os testes anteriores foram reexecutados após novos incrementos.

---

# Próxima prática

Na próxima aula prática, utilizaremos a mesma API e o Postman para aplicar técnicas de seleção de casos:

```text
Particionamento de Equivalência
Análise de Valor Limite
Tabela de Decisão
Pairwise
```

A pergunta central será:

> **Quais dados devemos selecionar para testar uma API de forma sistemática?**

---

# Referências

## Projeto

**Menu-Back-End**

```text
https://github.com/Herysson/Menu-Back-End
```

## Documentação oficial do Postman

### Collections

```text
https://learning.postman.com/docs/collections/collections-overview/
```

### Environments e Variables

```text
https://learning.postman.com/docs/sending-requests/variables/managing-environments/
```

### Scripts

```text
https://learning.postman.com/docs/tests-and-scripts/write-scripts/intro-to-scripts/
```

### Test scripts

```text
https://learning.postman.com/docs/tests-and-scripts/write-scripts/test-scripts/
```

### Collection Runner

```text
https://learning.postman.com/docs/collections/running-collections/intro-to-collection-runs/
```

## Bibliografia da disciplina

- PRESSMAN, Roger S.; MAXIM, Bruce R. *Engenharia de software: uma abordagem profissional*. 9. ed. Porto Alegre: AMGH, 2021.
- MARTINS, Marcos Danilo Chiodi. *Testes de software*. 1. ed. Rio de Janeiro: SESES, 2016.
