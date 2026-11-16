# Aula Prática 11.1 – Teste de Validação Automatizado com Selenium WebDriver

## Objetivos

Nesta aula prática, utilizaremos **Java**, **JUnit** e **Selenium WebDriver** para automatizar testes de validação de uma aplicação Web.

A proposta é continuar a sequência das aulas anteriores:

```text
JUnit
↓
Teste de unidade

Postman
↓
Teste de integração / API

Selenium WebDriver
↓
Teste de validação pela interface do usuário
```

Ao final da atividade, você deverá ser capaz de:

- compreender o papel do Selenium WebDriver;
- relacionar requisitos e critérios de aceitação a testes automatizados;
- iniciar e encerrar uma sessão de navegador;
- localizar elementos de uma página Web;
- simular ações de um usuário;
- criar verificações utilizando JUnit;
- utilizar esperas explícitas;
- organizar vários testes em uma suíte executável;
- executar todos os testes automaticamente;
- compreender como a mesma automação pode apoiar testes de regressão;
- perceber quando a execução em diferentes ambientes começa a se aproximar do teste de sistema.

---

# 1. Onde estamos na estratégia de teste?

Nas aulas anteriores, ampliamos progressivamente o escopo do teste:

```text
Unidade
↓
Integração
↓
Validação
↓
Sistema
```

No **teste de validação**, o foco passa para os requisitos e para o comportamento percebido pelo usuário.

A pergunta principal deixa de ser:

> As partes do software conseguem trabalhar juntas?

e passa a ser:

> **O software se comporta da maneira esperada pelo usuário e pelos requisitos?**

Pressman e Maxim (2021) destacam que, na validação, o foco está nas ações visíveis ao usuário e nas saídas que ele consegue reconhecer.

Martins (2016) também destaca que os casos de validação podem ser definidos a partir da especificação de requisitos, casos de uso e cenários de utilização.

---

# 2. O que é Selenium?

**Selenium** é um conjunto de ferramentas voltado à automação de navegadores Web.

Nesta aula utilizaremos especificamente:

```text
Selenium WebDriver
```

O WebDriver permite que um programa controle um navegador real.

Por exemplo:

```text
Java
 ↓
Selenium WebDriver
 ↓
Chrome
 ↓
Página Web
```

Nosso código poderá:

- abrir uma página;
- localizar campos;
- digitar textos;
- clicar em botões;
- navegar entre páginas;
- consultar informações apresentadas;
- verificar resultados.

## Documentação oficial

```text
https://www.selenium.dev/documentation/webdriver/
```

---

# 3. Selenium não define o nível do teste

É importante observar:

> **Selenium é uma ferramenta de automação. O objetivo do teste é que determina o nível de teste.**

Nesta prática utilizaremos Selenium para verificar requisitos pela interface do usuário.

Exemplo:

```text
REQUISITO
Usuário válido deve conseguir realizar login.

        ↓

SELENIUM
abre a página
preenche usuário
preenche senha
clica em Login

        ↓

JUNIT
verifica o resultado esperado
```

Nesse contexto, estamos utilizando Selenium principalmente como apoio ao **teste de validação**.

Mais adiante veremos que executar os mesmos testes em diferentes navegadores e ambientes também pode fornecer evidências relacionadas ao **teste de sistema**.

---

# 4. Sistema utilizado na prática

Utilizaremos o ambiente demonstrativo:

**SauceDemo / Swag Labs**

```text
https://www.saucedemo.com/
```

Esse sistema é utilizado em exemplos e materiais da Sauce Labs para demonstração de automação de testes.

Credenciais públicas utilizadas nesta aula:

```text
Usuário: standard_user
Senha:   secret_sauce
```

> Esta aplicação é externa e requer conexão com a Internet.

---

# 5. Especificação didática

Para esta aula, adotaremos os seguintes requisitos **didáticos**.

Eles foram definidos para orientar a atividade e não representam uma especificação oficial do SauceDemo.

## RF01 – Login válido

> O sistema deve permitir acesso ao catálogo quando forem informadas credenciais válidas.

Critério de aceitação:

```text
DADO que o usuário está na página de login
QUANDO informar credenciais válidas
E clicar em Login
ENTÃO o catálogo de produtos deve ser apresentado.
```

---

## RF02 – Login inválido

> O sistema deve impedir o acesso quando forem informadas credenciais inválidas.

Critério de aceitação:

```text
DADO que o usuário está na página de login
QUANDO informar credenciais inválidas
E clicar em Login
ENTÃO o acesso ao catálogo deve ser impedido
E uma mensagem de erro deve ser apresentada.
```

---

## RF03 – Adicionar produto ao carrinho

> Um usuário autenticado deve conseguir adicionar um produto ao carrinho.

Critério de aceitação:

```text
DADO que o usuário está autenticado
QUANDO adicionar um produto ao carrinho
ENTÃO o carrinho deve indicar que possui 1 item.
```

---

# 6. Da especificação para os casos de teste

Podemos organizar inicialmente:

| Caso | Requisito | Entrada/Ação | Resultado esperado |
|---|---|---|---|
| CT01 | RF01 | `standard_user` + `secret_sauce` | Catálogo apresentado |
| CT02 | RF02 | credenciais inválidas | Acesso negado e mensagem exibida |
| CT03 | RF03 | adicionar um produto | Carrinho indica `1` item |

Essa relação é importante:

```text
Requisito
   ↓
Critério de aceitação
   ↓
Caso de teste
   ↓
Automação
   ↓
Evidência PASS / FAIL
```

---

# 7. Ferramentas necessárias

Utilizaremos:

- Java 17 ou superior;
- Maven;
- JUnit 6;
- Selenium WebDriver;
- Google Chrome;
- uma IDE Java.

Versões utilizadas como referência na elaboração deste material:

```text
Selenium Java: 4.52.0
JUnit:         6.1.3
```

A versão atual do Selenium pode ser consultada em:

```text
https://www.selenium.dev/downloads/
```

---

# 8. Selenium Manager

Em versões atuais do Selenium, normalmente **não é necessário baixar manualmente o ChromeDriver**.

O Selenium utiliza o **Selenium Manager** para localizar ou obter o driver adequado ao navegador.

Assim, podemos iniciar o Chrome simplesmente com:

```java
WebDriver driver = new ChromeDriver();
```

O Selenium Manager cuida da configuração do driver quando necessário.

## Documentação

```text
https://www.selenium.dev/documentation/selenium_manager/
```

---

# 9. Criando o projeto Maven

Crie um projeto Maven.

Sugestão:

```text
teste-validacao-selenium
```

Estrutura esperada:

```text
teste-validacao-selenium
│
├── pom.xml
│
└── src
    └── test
        └── java
            └── br
                └── edu
                    └── ufn
                        └── validacao
                            └── SauceDemoTest.java
```

---

# 10. Configurando o `pom.xml`

Utilize:

```xml
<dependencies>

        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>4.52.0</version>
            <scope>compile</scope>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>6.1.3</version>
            <scope>test</scope>
        </dependency>

</dependencies>
```

Atualize as dependências Maven na IDE.

---

# 11. Primeiro contato com o WebDriver

Crie:

```text
SauceDemoTest.java
```

Conteúdo inicial:

```java
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

class SauceDemoTest {

    @Test
    void abrirPagina() throws InterruptedException {

        WebDriver driver = new ChromeDriver();

        driver.get("https://www.saucedemo.com/");

        // Aguarda 5 segundos para visualizar a página aberta
        Thread.sleep(5000);

        driver.quit();
    }
}
```

Execute.

O comportamento esperado é:

```text
JUnit
 ↓
Selenium
 ↓
Chrome é iniciado
 ↓
SauceDemo é aberto
 ↓
Chrome é encerrado
```

---

# 12. Entendendo o primeiro código

## Criando o WebDriver

```java
WebDriver driver = new ChromeDriver();
```

Cria uma sessão de automação com o Chrome.

---

## Navegando

```java
driver.get("https://www.saucedemo.com/");
```

Solicita que o navegador acesse a página.

---

## Encerrando

```java
driver.quit();
```

Finaliza a sessão e fecha o navegador.

> Encerrar a sessão ao término dos testes é importante para evitar processos de navegador abertos desnecessariamente.

---

# 13. Preparando e finalizando cada teste

Não queremos repetir a criação e o encerramento do navegador em todos os métodos.

Utilizaremos:

```text
@BeforeEach
```

e:

```text
@AfterEach
```

Atualize a classe:

```java
package br.edu.ufn.validacao;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

class SauceDemoTest {

    private WebDriver driver;

    @BeforeEach
    void iniciar() {
        driver = new ChromeDriver();
        driver.manage().window().maximize();
    }

    @AfterEach
    void finalizar() {
        if (driver != null) {
            driver.quit();
        }
    }

    @Test
    void abrirPagina() {
        driver.get("https://www.saucedemo.com/");
    }
}
```

Agora:

```text
@BeforeEach
    ↓
abre navegador
    ↓
@Test
    ↓
executa cenário
    ↓
@AfterEach
    ↓
fecha navegador
```

---

# 14. Localizando elementos

Para interagir com uma página, Selenium precisa localizar os elementos do HTML.

A classe utilizada é:

```java
By
```

Algumas possibilidades:

```java
By.id("user-name")
```

```java
By.name("user-name")
```

```java
By.cssSelector("#user-name")
```

```java
By.cssSelector("[data-test='username']")
```

```java
By.xpath("//input[@id='user-name']")
```

Nesta aula utilizaremos preferencialmente:

```text
ID
```

ou:

```text
CSS Selector
```

---

# 15. Como descobrir um seletor?

No navegador:

1. clique com o botão direito sobre um elemento;
2. escolha **Inspecionar**;
3. observe o HTML.

Por exemplo, o campo de usuário possui informações como:

```html
<input
    data-test="username"
    id="user-name"
    name="user-name"
    type="text">
```

Podemos localizá-lo por:

```java
By.id("user-name")
```

ou:

```java
By.cssSelector("[data-test='username']")
```

---

# 16. Localizando um `WebElement`

Exemplo:

```java
WebElement campoUsuario =
        driver.findElement(By.id("user-name"));
```

Temos:

```text
driver
  ↓
procura elemento
  ↓
By.id("user-name")
  ↓
WebElement
```

`WebElement` representa um elemento da página com o qual podemos interagir.

---

# 17. Ações comuns

## Digitar

```java
campoUsuario.sendKeys("standard_user");
```

---

## Clicar

```java
botao.click();
```

---

## Limpar

```java
campoUsuario.clear();
```

---

## Ler texto

```java
String texto = elemento.getText();
```

Essas operações permitem reproduzir ações que um usuário realizaria no navegador.

---

# 18. CT01 – Login válido

Agora implementaremos:

```text
RF01
↓
CT01
↓
Login com credenciais válidas
```

Adicione os imports:

```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;

import static org.junit.jupiter.api.Assertions.assertTrue;

class SauceDemoTest {

    private WebDriver driver;

    @BeforeEach
    void iniciar() {
        driver = new ChromeDriver();
        driver.manage().window().maximize();
    }

    @AfterEach
    void finalizar() {
        if (driver != null) {
            driver.quit();
        }
    }

    @Test
    void abrirPagina() throws InterruptedException {

        driver.get("https://www.saucedemo.com/");

        // Aguarda 5 segundos para visualizar a página
        Thread.sleep(5000);
    }

    @Test
    @DisplayName("RF01 - Login válido deve apresentar o catálogo")
    void loginValidoDeveApresentarCatalogo() throws InterruptedException {

        // Abre a página
        driver.get("https://www.saucedemo.com/");

        Thread.sleep(2000);

        // Localiza os elementos
        WebElement usuario =
                driver.findElement(By.id("user-name"));

        WebElement senha =
                driver.findElement(By.id("password"));

        WebElement login =
                driver.findElement(By.id("login-button"));

        // Digita o usuário
        usuario.sendKeys("standard_user");

        Thread.sleep(1500);

        // Digita a senha
        senha.sendKeys("secret_sauce");

        Thread.sleep(1500);

        // Clica no botão Login
        login.click();

        // Aguarda para que os alunos vejam o catálogo
        Thread.sleep(3000);

        // Verifica se o usuário foi direcionado
        // para a página de produtos
        assertTrue(
                driver.getCurrentUrl().contains("inventory.html")
        );

        // Mantém a página aberta mais um pouco
        Thread.sleep(3000);
    }
}
```

Execute o teste.

---

# 19. O que o CT01 realmente fez?

Automatizamos:

```text
Abrir SauceDemo
      ↓
Localizar usuário
      ↓
Digitar standard_user
      ↓
Localizar senha
      ↓
Digitar secret_sauce
      ↓
Clicar Login
      ↓
Observar resultado
      ↓
Comparar com esperado
```

A parte:

```java
assertTrue(
    driver.getCurrentUrl().contains("inventory.html")
);
```

é especialmente importante.

Sem uma assertiva, teríamos apenas:

```text
automação de ações
```

Com uma assertiva, temos:

```text
execução
   +
resultado esperado
   =
teste automatizado
```

---

# 20. Validando algo visível ao usuário

Podemos tornar o teste mais representativo da validação.

Depois do login, a página apresenta o catálogo com o título:

```text
Products
```

Adicione:

```java
WebElement titulo =
        driver.findElement(By.cssSelector(".title"));

assertEquals(
        "Products",
        titulo.getText()
);
```

O teste completo fica:

```java
@Test
@DisplayName("RF01 - Login válido deve apresentar o catálogo")
void loginValidoDeveApresentarCatalogo() {

    driver.get("https://www.saucedemo.com/");

    driver.findElement(By.id("user-name"))
          .sendKeys("standard_user");

    driver.findElement(By.id("password"))
          .sendKeys("secret_sauce");

    driver.findElement(By.id("login-button"))
          .click();

    assertTrue(
        driver.getCurrentUrl().contains("inventory.html")
    );

    WebElement titulo =
            driver.findElement(By.cssSelector(".title"));

    assertEquals("Products", titulo.getText());
}
```

Agora verificamos uma saída efetivamente reconhecível pelo usuário.

---

# 21. O problema do tempo

Aplicações Web nem sempre respondem instantaneamente.

Podemos ter:

```text
Selenium clica
      ↓
JavaScript ainda está processando
      ↓
Selenium procura o próximo elemento
      ↓
elemento ainda não apareceu
      ↓
teste falha
```

Esse tipo de problema pode gerar testes instáveis.

---

# 22. Evite `Thread.sleep()`

Uma solução aparentemente simples seria:

```java
Thread.sleep(5000);
```

Mas isso significa:

> espere sempre 5 segundos.

Problemas:

- talvez 5 segundos não sejam suficientes;
- talvez a página fique pronta em 200 ms;
- a suíte fica desnecessariamente lenta;
- o teste pode continuar instável.

O Selenium recomenda estratégias de espera adequadas em vez de pausas fixas.

---

# 23. Esperas explícitas

Vamos utilizar:

```java
WebDriverWait
```

Adicione:

```java
import java.time.Duration;

import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
```

Na classe:

```java
private WebDriver driver;
private WebDriverWait wait;
```

No `@BeforeEach`:

```java
@BeforeEach
void iniciar() {

    driver = new ChromeDriver();

    driver.manage().window().maximize();

    wait = new WebDriverWait(
        driver,
        Duration.ofSeconds(10)
    );
}
```

---

# 24. Esperando uma condição

Depois do login:

```java
WebElement titulo = wait.until(
    ExpectedConditions.visibilityOfElementLocated(
        By.cssSelector(".title")
    )
);
```

Significa:

```text
aguarde
↓
até que
↓
.title
↓
esteja visível
```

Se isso acontecer antes de 10 segundos, o teste continua imediatamente.

Se não acontecer dentro do limite, ocorre timeout.

---

# 25. CT01 com espera explícita

Atualize:

```java
@Test
@DisplayName("RF01 - Login válido deve apresentar o catálogo")
void loginValidoDeveApresentarCatalogo() {

    driver.get("https://www.saucedemo.com/");

    driver.findElement(By.id("user-name"))
          .sendKeys("standard_user");

    driver.findElement(By.id("password"))
          .sendKeys("secret_sauce");

    driver.findElement(By.id("login-button"))
          .click();

    WebElement titulo = wait.until(
        ExpectedConditions.visibilityOfElementLocated(
            By.cssSelector(".title")
        )
    );

    assertEquals("Products", titulo.getText());

    assertTrue(
        driver.getCurrentUrl().contains("inventory.html")
    );
}
```

## Documentação

```text
https://www.selenium.dev/documentation/webdriver/waits/
```

> Evite misturar esperas implícitas e explícitas no mesmo teste.

---

# 26. CT02 – Login inválido

Agora testaremos:

```text
RF02
```

Adicione:

```java
@Test
@DisplayName("RF02 - Login inválido deve ser rejeitado")
void loginInvalidoDeveSerRejeitado() {

    driver.get("https://www.saucedemo.com/");

    driver.findElement(By.id("user-name"))
          .sendKeys("usuario_invalido");

    driver.findElement(By.id("password"))
          .sendKeys("senha_invalida");

    driver.findElement(By.id("login-button"))
          .click();

    WebElement erro = wait.until(
        ExpectedConditions.visibilityOfElementLocated(
            By.cssSelector(".error-message-container")
        )
    );

    assertTrue(erro.isDisplayed());

    assertTrue(
        !erro.getText().isBlank()
    );

    assertTrue(
        !driver.getCurrentUrl().contains("inventory.html")
    );
}
```

---

# 27. Por que não verificar somente a mensagem?

Nosso requisito didático diz:

```text
credenciais inválidas
↓
não devem permitir acesso
+
devem apresentar uma mensagem
```

Portanto verificamos duas coisas:

```text
Mensagem de erro apareceu?
```

e:

```text
Usuário permaneceu fora do catálogo?
```

Um bom teste deve estar relacionado ao comportamento especificado.

---

# 28. Evitando repetição

Nos próximos testes precisaremos fazer login novamente.

Em vez de repetir:

```java
driver.findElement(...)
driver.findElement(...)
driver.findElement(...)
```

podemos criar um método auxiliar:

```java
private void fazerLogin(
        String usuario,
        String senha) {

    driver.get("https://www.saucedemo.com/");

    driver.findElement(By.id("user-name"))
          .sendKeys(usuario);

    driver.findElement(By.id("password"))
          .sendKeys(senha);

    driver.findElement(By.id("login-button"))
          .click();
}
```

Agora podemos utilizar:

```java
fazerLogin(
    "standard_user",
    "secret_sauce"
);
```

---

# 29. CT03 – Adicionar produto ao carrinho

Implementaremos:

```text
RF03
```

Teste:

```java
@Test
@DisplayName("RF03 - Produto adicionado deve aparecer no carrinho")
void adicionarProdutoDeveAtualizarCarrinho() {

    fazerLogin(
        "standard_user",
        "secret_sauce"
    );

    WebElement titulo = wait.until(
        ExpectedConditions.visibilityOfElementLocated(
            By.cssSelector(".title")
        )
    );

    assertEquals("Products", titulo.getText());

    driver.findElement(
        By.cssSelector(
            "[data-test='add-to-cart-sauce-labs-backpack']"
        )
    ).click();

    WebElement quantidade = wait.until(
        ExpectedConditions.visibilityOfElementLocated(
            By.cssSelector(".shopping_cart_badge")
        )
    );

    assertEquals(
        "1",
        quantidade.getText()
    );
}
```

---

# 30. O que foi validado no CT03?

O fluxo executado foi:

```text
Login
  ↓
Catálogo
  ↓
Adicionar Sauce Labs Backpack
  ↓
Carrinho
  ↓
Quantidade = 1?
```

Do ponto de vista do usuário:

> Eu cliquei para adicionar um produto. O sistema apresentou evidência de que o produto foi adicionado?

Essa é uma pergunta típica de validação.

---

# 31. Classe completa até aqui

```java
package br.edu.ufn.validacao;

import java.time.Duration;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;

import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

class SauceDemoTest {

    private static final String BASE_URL =
            "https://www.saucedemo.com/";

    private WebDriver driver;
    private WebDriverWait wait;

    @BeforeEach
    void iniciar() {

        driver = new ChromeDriver();

        driver.manage()
              .window()
              .maximize();

        wait = new WebDriverWait(
            driver,
            Duration.ofSeconds(10)
        );
    }

    @AfterEach
    void finalizar() {

        if (driver != null) {
            driver.quit();
        }
    }

    private void fazerLogin(
            String usuario,
            String senha) {

        driver.get(BASE_URL);

        driver.findElement(By.id("user-name"))
              .sendKeys(usuario);

        driver.findElement(By.id("password"))
              .sendKeys(senha);

        driver.findElement(By.id("login-button"))
              .click();
    }

    @Test
    @DisplayName(
        "RF01 - Login válido deve apresentar o catálogo"
    )
    void loginValidoDeveApresentarCatalogo() {

        fazerLogin(
            "standard_user",
            "secret_sauce"
        );

        WebElement titulo = wait.until(
            ExpectedConditions.visibilityOfElementLocated(
                By.cssSelector(".title")
            )
        );

        assertEquals(
            "Products",
            titulo.getText()
        );

        assertTrue(
            driver.getCurrentUrl()
                  .contains("inventory.html")
        );
    }

    @Test
    @DisplayName(
        "RF02 - Login inválido deve ser rejeitado"
    )
    void loginInvalidoDeveSerRejeitado() {

        fazerLogin(
            "usuario_invalido",
            "senha_invalida"
        );

        WebElement erro = wait.until(
            ExpectedConditions.visibilityOfElementLocated(
                By.cssSelector(
                    ".error-message-container"
                )
            )
        );

        assertTrue(erro.isDisplayed());

        assertTrue(
            !erro.getText().isBlank()
        );

        assertTrue(
            !driver.getCurrentUrl()
                   .contains("inventory.html")
        );
    }

    @Test
    @DisplayName(
        "RF03 - Produto adicionado deve aparecer no carrinho"
    )
    void adicionarProdutoDeveAtualizarCarrinho() {

        fazerLogin(
            "standard_user",
            "secret_sauce"
        );

        WebElement titulo = wait.until(
            ExpectedConditions.visibilityOfElementLocated(
                By.cssSelector(".title")
            )
        );

        assertEquals(
            "Products",
            titulo.getText()
        );

        driver.findElement(
            By.cssSelector(
                "[data-test='add-to-cart-sauce-labs-backpack']"
            )
        ).click();

        WebElement quantidade = wait.until(
            ExpectedConditions.visibilityOfElementLocated(
                By.cssSelector(
                    ".shopping_cart_badge"
                )
            )
        );

        assertEquals(
            "1",
            quantidade.getText()
        );
    }
}
```

---

# 32. Executando um teste

Na IDE, execute um método individual:

```text
loginValidoDeveApresentarCatalogo()
```

O navegador será aberto automaticamente.

Depois:

```text
PASS
```

ou:

```text
FAIL
```

será apresentado pelo JUnit.

---

# 33. Executando todos os testes

Agora execute a classe:

```text
SauceDemoTest
```

Os três testes serão executados.

Temos:

```text
CT01
↓
abre navegador
↓
executa
↓
fecha navegador

CT02
↓
abre navegador
↓
executa
↓
fecha navegador

CT03
↓
abre navegador
↓
executa
↓
fecha navegador
```

---

# 34. Executando pelo Maven

Também podemos executar todos os testes pelo terminal:

```bash
mvn test
```

O Maven executará os testes encontrados em:

```text
src/test/java
```

Essa é uma etapa importante para automação.

Agora o teste não depende obrigatoriamente de alguém clicar no botão:

```text
Run
```

da IDE.

---

# 35. Relatórios do Maven

Após:

```bash
mvn test
```

verifique:

```text
target/surefire-reports/
```

Nesse diretório são armazenados resultados gerados pelo Maven Surefire.

Exemplo:

```text
target/
└── surefire-reports/
    ├── TEST-br.edu.ufn.validacao.SauceDemoTest.xml
    └── br.edu.ufn.validacao.SauceDemoTest.txt
```

Esses arquivos podem ser utilizados como evidência da execução automatizada.

---

# 36. O que significa automatizar?

Sem automação:

```text
Abrir navegador
↓
digitar usuário
↓
digitar senha
↓
clicar
↓
observar
↓
anotar resultado
```

Para cada execução.

Com Selenium:

```text
mvn test
   ↓
JUnit
   ↓
Selenium
   ↓
abre navegador
   ↓
executa ações
   ↓
compara resultados
   ↓
PASS / FAIL
   ↓
relatório
```

---

# 37. Automação não elimina o projeto de teste

Selenium consegue executar ações automaticamente.

Mas ele não decide sozinho:

- qual requisito é importante;
- qual cenário deve ser testado;
- qual entrada deve ser utilizada;
- qual resultado é correto;
- qual risco merece prioridade.

Portanto:

```text
Projeto de teste
      +
Selenium
      =
execução automatizada
```

---

# 38. Automação e regressão

Imagine que os três testes estejam aprovados:

```text
RF01 → PASS
RF02 → PASS
RF03 → PASS
```

A equipe altera a tela de login.

Depois da alteração, executamos novamente:

```bash
mvn test
```

Resultado:

```text
RF01 → PASS
RF02 → FAIL
RF03 → PASS
```

O teste automatizado revelou que um comportamento anteriormente funcionando deixou de funcionar.

Isso pode indicar uma:

```text
REGRESSÃO
```

---

# 39. Selenium e teste de sistema

Até aqui nosso objetivo principal foi:

> validar requisitos pela interface do usuário.

Agora imagine executar a mesma suíte em:

```text
Chrome
Firefox
Edge
```

Temos:

```text
Chrome  → PASS
Firefox → PASS
Edge    → FAIL
```

A funcionalidade pode estar correta, mas existir um problema relacionado ao ambiente ou à compatibilidade.

Isso começa a aproximar nossa análise do **teste de sistema**, no qual o ambiente completo passa a fazer parte do problema de teste.

---

# 40. Executando com Firefox

Se o Firefox estiver instalado, troque:

```java
import org.openqa.selenium.chrome.ChromeDriver;
```

por:

```java
import org.openqa.selenium.firefox.FirefoxDriver;
```

e:

```java
driver = new ChromeDriver();
```

por:

```java
driver = new FirefoxDriver();
```

Execute novamente.

O Selenium Manager também pode gerenciar o driver correspondente.

---

# 41. Executando com Edge

No Microsoft Edge:

```java
import org.openqa.selenium.edge.EdgeDriver;
```

Depois:

```java
driver = new EdgeDriver();
```

Agora podemos produzir uma pequena matriz:

| Teste | Chrome | Firefox | Edge |
|---|---|---|---|
| RF01 – Login válido | PASS | PASS | PASS |
| RF02 – Login inválido | PASS | PASS | PASS |
| RF03 – Carrinho | PASS | PASS | PASS |

Se houver diferenças, investigue se estão associadas ao:

- requisito;
- código;
- navegador;
- versão;
- ambiente.

---

# 42. Selenium Grid

Quando uma equipe precisa executar muitos testes em vários navegadores, sistemas operacionais ou máquinas, pode utilizar:

```text
Selenium Grid
```

Exemplo:

```text
                ┌─ Chrome / Windows
Testes ─ Grid ──┼─ Firefox / Linux
                └─ Edge / Windows
```

Nesta primeira prática **não configuraremos o Grid**.

Nosso objetivo é compreender primeiro o WebDriver e a automação local.

## Documentação

```text
https://www.selenium.dev/documentation/grid/
```

---

# 43. O que não abordaremos nesta primeira prática?

Para manter a aula simples, não utilizaremos ainda:

- Page Object Model;
- Selenium Grid;
- execução paralela;
- Docker;
- CI/CD;
- testes visuais;
- execução headless;
- relatórios avançados;
- serviços de nuvem para testes.

Esses recursos são importantes em projetos maiores, mas não são necessários para compreender os fundamentos desta aula.

---

# 44. Atividade dos alunos

Crie um novo requisito didático:

## RF04 – Remover produto

> Um usuário deve conseguir remover do carrinho um produto previamente adicionado.

Defina antes de programar:

```text
Critério de aceitação
```

e:

```text
Caso de teste
```

Depois implemente o teste utilizando Selenium.

O teste deverá:

1. realizar login;
2. adicionar o `Sauce Labs Backpack`;
3. confirmar que o carrinho possui `1` item;
4. remover o produto;
5. verificar o resultado esperado.

> Não copie apenas ações. Defina primeiro qual evidência demonstrará que o requisito foi atendido.

---

# 45. Atividade – rastreabilidade

Complete:

| Requisito | Caso de teste | Evidência esperada | Resultado |
|---|---|---|---|
| RF01 | CT01 | Catálogo `Products` | |
| RF02 | CT02 | Erro apresentado e acesso negado | |
| RF03 | CT03 | Badge do carrinho = `1` | |
| RF04 | CT04 | A definir pelo aluno | |

Depois execute:

```bash
mvn test
```

e registre:

```text
PASS
```

ou:

```text
FAIL
```

---

# 46. Desafio – teste em diferentes navegadores

Execute a suíte em pelo menos dois navegadores.

Por exemplo:

```text
Chrome
+
Firefox
```

Registre:

| Caso | Chrome | Firefox |
|---|---|---|
| CT01 | | |
| CT02 | | |
| CT03 | | |
| CT04 | | |

Responda:

> Se um caso passa no Chrome e falha no Firefox, estamos necessariamente diante de um requisito funcional incorreto?

Justifique.

---

# 47. Questões para discussão

1. Qual a diferença entre automatizar ações e automatizar um teste?
2. Por que uma assertiva é importante?
3. O que o `WebDriver` representa?
4. Qual a função de `driver.get()`?
5. O que é um `WebElement`?
6. Para que serve a classe `By`?
7. Qual a diferença entre `By.id()` e `By.cssSelector()`?
8. Por que devemos evitar `Thread.sleep()`?
9. Qual a finalidade de uma espera explícita?
10. Por que utilizamos `@BeforeEach`?
11. Por que utilizamos `@AfterEach`?
12. Qual a relação entre requisito e caso de teste?
13. Como Selenium pode apoiar testes de regressão?
14. Por que executar o mesmo teste em diferentes navegadores pode revelar problemas que não aparecem em um único ambiente?
15. Selenium é, por definição, uma ferramenta de teste de validação? Explique.

---

# 48. Checklist

Antes de finalizar:

- [ ] Java está instalado.
- [ ] Maven está configurado.
- [ ] Selenium foi adicionado ao `pom.xml`.
- [ ] JUnit foi adicionado ao `pom.xml`.
- [ ] O primeiro navegador foi aberto pelo Selenium.
- [ ] O navegador é encerrado após cada teste.
- [ ] O campo de usuário foi localizado.
- [ ] O campo de senha foi localizado.
- [ ] O botão Login foi localizado.
- [ ] CT01 foi implementado.
- [ ] CT02 foi implementado.
- [ ] CT03 foi implementado.
- [ ] Foram utilizadas assertivas.
- [ ] Foi utilizada espera explícita.
- [ ] `Thread.sleep()` não foi utilizado como estratégia principal de sincronização.
- [ ] Todos os testes foram executados em conjunto.
- [ ] `mvn test` foi executado.
- [ ] Os relatórios do Surefire foram localizados.
- [ ] RF04 foi desenvolvido pelo aluno.
- [ ] A matriz de rastreabilidade foi preenchida.
- [ ] A suíte foi executada novamente após alterações.
- [ ] Pelo menos dois navegadores foram comparados no desafio.

---

# 49. Resumo

Nesta prática utilizamos:

```text
Requisitos
   ↓
Critérios de aceitação
   ↓
Casos de teste
   ↓
JUnit
   ↓
Selenium WebDriver
   ↓
Navegador real
   ↓
Ações do usuário
   ↓
Resultados visíveis
   ↓
Assertivas
   ↓
PASS / FAIL
```

O ponto central é:

> **Selenium automatiza a interação com o navegador, mas o teste continua dependendo de requisitos, casos de teste e resultados esperados bem definidos.**

Na validação, utilizamos a automação para obter evidências de que o software se comporta da maneira esperada pelo usuário.

Quando passamos a considerar diferentes navegadores, sistemas operacionais, dispositivos e configurações, começamos a ampliar o escopo em direção ao **teste de sistema**.

---

# Referências

## Bibliografia da disciplina

PRESSMAN, Roger S.; MAXIM, Bruce R. **Engenharia de software: uma abordagem profissional**. 9. ed. Porto Alegre: AMGH, 2021.

MARTINS, Marcos Danilo Chiodi. **Testes de software**. 1. ed. Rio de Janeiro: SESES, 2016.

---

## Selenium

Documentação oficial:

```text
https://www.selenium.dev/documentation/
```

WebDriver:

```text
https://www.selenium.dev/documentation/webdriver/
```

Primeiro script:

```text
https://www.selenium.dev/documentation/webdriver/getting_started/
```

Estratégias de espera:

```text
https://www.selenium.dev/documentation/webdriver/waits/
```

Downloads e versões:

```text
https://www.selenium.dev/downloads/
```

Selenium Grid:

```text
https://www.selenium.dev/documentation/grid/
```

---

## JUnit

JUnit User Guide:

```text
https://docs.junit.org/6.1.3/
```

---

## SauceDemo / Sauce Labs

Sistema demonstrativo:

```text
https://www.saucedemo.com/
```

Documentação de exemplo com Selenium:

```text
https://docs.saucelabs.com/web-apps/automated-testing/selenium/
```

> A aplicação SauceDemo é utilizada nesta aula apenas como ambiente didático de demonstração. Os requisitos RF01–RF04 foram definidos especificamente para esta atividade.
