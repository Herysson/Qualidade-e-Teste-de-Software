# Aula Prática 10.1 – Primeiros Testes de Integração com Postman

## Objetivos

Nesta aula prática, utilizaremos o projeto **Menu-Back-End** e a ferramenta **Postman** para compreender, de forma simples, como realizar testes de integração em uma API REST.

Ao final da atividade, você deverá ser capaz de:

- executar a aplicação `Menu-Back-End`;
- enviar requisições HTTP utilizando o Postman;
- organizar requisições em uma Collection;
- utilizar variáveis de ambiente;
- criar verificações automatizadas sobre as respostas;
- executar um fluxo CRUD completo;
- compreender o que está sendo integrado durante os testes;
- compreender, de forma prática, a ideia de **integração incremental**.

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

# 2. O que estamos testando?

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

O resultado percorre o caminho inverso até ser convertido novamente em uma resposta HTTP.

Portanto, uma falha pode estar:

- na requisição HTTP;
- na conversão do JSON;
- no Controller;
- na persistência;
- no mapeamento JPA;
- na comunicação com o banco;
- na construção da resposta.

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

Configure o arquivo:

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

# 5. Primeiro teste manual

Abra o Postman e execute:

```http
GET http://localhost:8080/produtos
```

Observe:

- código de status;
- corpo da resposta;
- formato JSON;
- tempo de resposta.

Se a aplicação estiver funcionando corretamente, deverá ser retornada a lista de produtos cadastrados.

## Reflexão

Se esse teste falhar, a causa obrigatoriamente está no Controller?

**Não.**

A falha pode estar em qualquer ponto da integração:

```text
Postman
   ↓
Spring Boot
   ↓
JPA
   ↓
MySQL
```

Esse é um dos motivos pelos quais testes de integração são importantes.

---

# 6. Criando uma Collection

No Postman:

1. acesse **Collections**;
2. clique em **Create Collection**;
3. crie a coleção:

```text
Menu Back-End - Testes de Integração
```

Dentro dela, crie inicialmente a pasta:

```text
01 - Fluxo CRUD
```

Ao final da aula, teremos algo semelhante a:

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

---

# 7. Criando um Environment

Crie um ambiente chamado:

```text
Menu - Local
```

Adicione:

| Variável | Valor |
|---|---|
| `baseUrl` | `http://localhost:8080` |

Selecione esse ambiente no Postman.

A partir de agora, utilize:

```text
{{baseUrl}}/produtos
```

em vez de:

```text
http://localhost:8080/produtos
```

Isso permite alterar o endereço da aplicação em apenas um lugar.

---

# 8. Primeiro teste automatizado

Na requisição:

```http
GET {{baseUrl}}/produtos
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

Adicione também:

```javascript
pm.test("A resposta deve ser JSON", function () {
    pm.response.to.be.json;
});
```

E:

```javascript
pm.test("O corpo da resposta deve ser uma lista", function () {
    const body = pm.response.json();
    pm.expect(body).to.be.an("array");
});
```

Execute novamente a requisição.

O Postman deverá mostrar o resultado de cada verificação:

```text
PASS
```

ou:

```text
FAIL
```

---

# 9. Executar não é o mesmo que testar

Enviar uma requisição e observar visualmente a resposta é útil, mas um teste deve possuir um **resultado esperado**.

Podemos representar a ideia assim:

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

Por exemplo:

```javascript
pm.test("O corpo da resposta deve ser uma lista", function () {
    const body = pm.response.json();
    pm.expect(body).to.be.an("array");
});
```

Nesse caso, estamos explicitando uma condição que deve ser verdadeira para o teste ser considerado aprovado.

---

# 10. Criando um produto de teste

Crie uma requisição:

```text
POST - Criar produto
```

Antes da requisição, abra:

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

Isso cria um nome diferente a cada execução.

Configure:

```http
POST {{baseUrl}}/produtos
```

No Body, selecione:

```text
raw → JSON
```

e utilize:

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

Execute a requisição.

---

# 11. Verificando a criação

No **Post-response**, adicione:

```javascript
pm.test("A criação deve retornar sucesso", function () {
    pm.response.to.be.success;
});
```

Caso a API devolva o produto criado, podemos verificar o nome:

```javascript
pm.test("O nome retornado deve ser o nome enviado", function () {
    const body = pm.response.json();

    pm.expect(body.nome)
        .to.eql(pm.environment.get("nomeProdutoTeste"));
});
```

Se a resposta possuir um campo `id`, armazene-o:

```javascript
const body = pm.response.json();

if (body.id !== undefined && body.id !== null) {
    pm.environment.set("produtoId", body.id);
}
```

---

# 12. Localizando o produto criado

Se o endpoint `POST` não devolver o identificador, podemos localizá-lo pela listagem.

Crie:

```text
GET - Localizar produto
```

Requisição:

```http
GET {{baseUrl}}/produtos
```

No Post-response:

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

Agora podemos utilizar:

```text
{{produtoId}}
```

nas demais requisições.

---

# 13. Buscando o produto pelo ID

Crie:

```text
GET - Buscar produto por ID
```

Requisição:

```http
GET {{baseUrl}}/produtos/{{produtoId}}
```

Adicione:

```javascript
pm.test("A consulta deve retornar sucesso", function () {
    pm.response.to.be.success;
});
```

E:

```javascript
pm.test("O produto retornado deve possuir o ID esperado", function () {
    const body = pm.response.json();

    pm.expect(String(body.id))
        .to.eql(String(pm.environment.get("produtoId")));
});
```

Também podemos verificar o nome:

```javascript
pm.test("O produto deve manter o nome cadastrado", function () {
    const body = pm.response.json();

    pm.expect(body.nome)
        .to.eql(pm.environment.get("nomeProdutoTeste"));
});
```

---

# 14. O que foi integrado até aqui?

Observe o fluxo:

```text
POST
 ↓
Spring Boot
 ↓
persistência
 ↓
MySQL
```

Depois:

```text
GET
 ↓
Spring Boot
 ↓
MySQL
 ↓
JSON
```

O dado criado por uma operação está sendo recuperado por outra.

Isso permite verificar a integração entre:

- API HTTP;
- aplicação;
- persistência;
- banco de dados;
- serialização da resposta.

---

# 15. Atualizando o produto

Crie:

```text
PUT - Atualizar produto
```

Requisição:

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

No Post-response:

```javascript
pm.test("A atualização deve retornar sucesso", function () {
    pm.response.to.be.success;
});
```

Execute a requisição.

---

# 16. Confirmando a persistência da atualização

Uma resposta de sucesso do `PUT` não demonstra, sozinha, que os dados foram persistidos corretamente.

Crie:

```text
GET - Confirmar atualização
```

Requisição:

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

E:

```javascript
pm.test("O produto deve estar indisponível", function () {
    const body = pm.response.json();

    pm.expect(body.disponibilidade).to.eql(false);
});
```

Agora estamos verificando:

```text
PUT
 ↓
persistência
 ↓
GET
 ↓
confirmação dos dados
```

---

# 17. Excluindo o produto

Crie:

```text
DELETE - Excluir produto
```

Requisição:

```http
DELETE {{baseUrl}}/produtos/{{produtoId}}
```

Adicione:

```javascript
pm.test("A exclusão deve retornar sucesso", function () {
    pm.response.to.be.success;
});
```

Execute.

---

# 18. Confirmando a exclusão

Crie:

```text
GET - Confirmar exclusão
```

Utilize:

```http
GET {{baseUrl}}/produtos
```

No Post-response:

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

Assim, não verificamos apenas se o `DELETE` retornou sucesso.

Também confirmamos que o registro realmente deixou de existir na persistência.

---

# 19. O fluxo completo

Ao final, temos:

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

Esse é o fluxo principal desta prática.

Ele exercita diferentes partes da aplicação trabalhando em conjunto.

---

# 20. Integração incremental

Uma forma importante de realizar testes de integração é integrar e testar o sistema **gradualmente**.

A ideia é evitar integrar tudo de uma vez e somente depois procurar os problemas.

Podemos visualizar:

```text
Novo elemento
      ↓
Integrar
      ↓
Testar
      ↓
Adicionar outro elemento
      ↓
Testar novamente
```

No `Menu-Back-End`, utilizaremos essa ideia de forma didática, acrescentando gradualmente operações ao fluxo de teste.

## Incremento 1 – Listagem

Começamos com:

```text
GET /produtos
```

Verificamos:

- a aplicação está acessível;
- o banco pode ser consultado;
- a resposta é JSON;
- a resposta possui a estrutura esperada.

```text
GET
 ↓
TESTE
```

## Incremento 2 – Criação

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

> O produto criado pela operação POST pode ser recuperado posteriormente?

## Incremento 3 – Busca por ID

Adicionamos:

```text
GET /produtos/{id}
```

Agora:

```text
POST
 ↓
obter ID
 ↓
GET /{id}
```

E também mantemos os testes anteriores.

## Incremento 4 – Atualização

Adicionamos:

```text
PUT /produtos/{id}
```

Executamos:

```text
POST
 ↓
GET
 ↓
PUT
 ↓
GET
```

A segunda consulta confirma se a atualização foi realmente persistida.

## Incremento 5 – Exclusão

Finalmente adicionamos:

```text
DELETE /produtos/{id}
```

Fluxo:

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

Agora temos o fluxo completo.

---

# 21. Por que trabalhar de forma incremental?

Considere:

```text
GET funcionando
POST funcionando
GET por ID funcionando
```

Adicionamos:

```text
PUT
```

e os testes começam a falhar.

Como o problema apareceu logo após esse incremento, temos uma indicação melhor sobre **onde investigar**.

Esse é um dos benefícios da integração incremental:

> Ao acrescentar componentes ou comportamentos gradualmente e testar a cada etapa, fica mais fácil identificar em qual incremento surgiu um problema.

Também é importante **reexecutar os testes anteriores**.

Exemplo:

```text
Antes do PUT:
GET → PASS
POST → PASS
GET ID → PASS
```

Após adicionar o PUT:

```text
GET → PASS
POST → PASS
GET ID → PASS
PUT → PASS
GET confirmação → PASS
```

Se um teste anterior passar a falhar após uma nova integração, pode ter ocorrido uma **regressão**.

Nesta prática, o objetivo é apenas compreender essa ideia. Teste de regressão será aprofundado em outro momento.

---

# 22. Resumo do cenário testado

Nesta prática construímos progressivamente:

```text
Cadastrar
   ↓
Consultar
   ↓
Atualizar
   ↓
Consultar novamente
   ↓
Excluir
   ↓
Confirmar exclusão
```

Ao longo desse fluxo, verificamos a interação entre:

```text
Postman
   ↓
HTTP
   ↓
Spring Boot
   ↓
persistência
   ↓
MySQL
```

O objetivo não foi apenas verificar cada endpoint isoladamente.

Também verificamos se operações diferentes conseguem trabalhar sobre os mesmos dados de forma consistente.

---

# 23. Atividade prática

Utilizando o projeto `Menu-Back-End`, construa no Postman uma Collection que realize o seguinte fluxo:

1. listar os produtos;
2. cadastrar um novo produto;
3. localizar e armazenar o identificador do produto criado;
4. buscar o produto pelo identificador;
5. alterar pelo menos dois campos;
6. consultar novamente o produto;
7. confirmar que as alterações foram persistidas;
8. excluir o produto;
9. confirmar que ele não aparece mais na listagem.

Cada etapa deverá possuir pelo menos uma verificação automatizada.

## Organização esperada

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

---

# 24. Questões para discussão

1. Por que esse conjunto pode ser considerado um teste de integração?
2. Se o `GET /produtos` falhar, o defeito obrigatoriamente está no Controller?
3. Por que utilizamos uma nova consulta depois do `PUT`?
4. Qual a vantagem de armazenar o ID do produto em uma variável?
5. Qual a diferença entre executar uma requisição e criar um teste para essa requisição?
6. O que significa realizar a integração de forma incremental?
7. Por que os testes anteriores devem ser executados novamente quando um novo incremento é adicionado?
8. Se todos os testes passarem, podemos garantir que a integração não possui nenhum defeito?

---

# 25. Checklist

Antes de finalizar, verifique:

- [ ] A aplicação Spring Boot está executando.
- [ ] O MySQL está configurado.
- [ ] A Collection foi criada.
- [ ] O Environment `Menu - Local` está selecionado.
- [ ] A variável `baseUrl` está configurada.
- [ ] O teste de listagem foi executado.
- [ ] Um produto de teste foi criado.
- [ ] O ID do produto foi armazenado.
- [ ] O produto foi consultado pelo ID.
- [ ] O produto foi atualizado.
- [ ] A atualização foi confirmada com uma nova consulta.
- [ ] O produto foi excluído.
- [ ] A exclusão foi confirmada.
- [ ] As requisições possuem verificações automatizadas.
- [ ] Os testes anteriores foram reexecutados ao adicionar novos passos.

---

# Próxima prática

Na próxima aula prática, utilizaremos a mesma API e o Postman para aplicar técnicas de seleção de casos de teste:

```text
Particionamento de Equivalência
Análise de Valor Limite
Tabela de Decisão
Pairwise
```

O objetivo será responder outra pergunta:

> **Quais dados devemos escolher para testar uma API de forma sistemática?**

---

# Referências

## Projeto utilizado

**Menu-Back-End**

```text
https://github.com/Herysson/Menu-Back-End
```

## Postman

Documentação oficial:

```text
https://learning.postman.com/
```

Recursos utilizados nesta prática:

- Collections;
- Environments;
- Variables;
- Pre-request Scripts;
- Post-response Scripts;
- `pm.test`;
- `pm.expect`.

## Bibliografia da disciplina

- PRESSMAN, Roger S.; MAXIM, Bruce R. *Engenharia de software: uma abordagem profissional*. 9. ed. Porto Alegre: AMGH, 2021.
- MARTINS, Marcos Danilo Chiodi. *Testes de software*. 1. ed. Rio de Janeiro: SESES, 2016.
