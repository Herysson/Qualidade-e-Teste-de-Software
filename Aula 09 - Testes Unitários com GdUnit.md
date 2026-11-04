# Tutorial – Testes Unitários no Godot com GdUnit4

Este tutorial apresenta um passo a passo para criar e executar testes automatizados em projetos desenvolvidos com **Godot**, utilizando **GDScript** e o framework **GdUnit4**.

O objetivo é aplicar, em um contexto de Jogos Digitais, os mesmos princípios estudados em testes unitários:

- testar uma unidade de software de forma isolada;
- definir casos de teste a partir do comportamento esperado;
- aplicar técnicas de teste funcional e estrutural;
- automatizar a execução dos casos de teste;
- interpretar falhas e resultados da suíte.

---

## 1. Criando o projeto no Godot

Abra o **Godot** e crie um novo projeto.

Exemplo:

```text
Nome do projeto: TestesUnitariosJogos
Renderer: Compatibility ou Forward+
```

Após criar o projeto, aguarde a abertura do editor.

Uma possível organização inicial será:

```text
res://
├── scripts/
├── test/
└── addons/
```

As pastas serão criadas ao longo do tutorial.

---

# 2. Instalando o GdUnit4

O **GdUnit4** é um plugin de testes para Godot que permite criar e executar testes automatizados diretamente no editor.

## 2.1 Instalação pela Asset Library

No editor do Godot:

1. abra a aba **AssetLib**;
2. pesquise por:

```text
GdUnit4
```

3. selecione o plugin **GdUnit4**;
4. clique em **Download**;
5. confirme os arquivos;
6. clique em **Install**.

Após a instalação, o plugin normalmente será colocado em:

```text
res://addons/gdunit4/
```

---

## 2.2 Ativando o plugin

Acesse:

```text
Project
→ Project Settings
→ Plugins
```

Localize **GdUnit4** e habilite o plugin.

Após a ativação, o editor deverá exibir as ferramentas do GdUnit4.

> É recomendável reiniciar o editor após a instalação do plugin, especialmente se o painel do GdUnit4 não aparecer corretamente.

---

# 3. Estrutura dos testes

Uma organização possível para o projeto é:

```text
res://
├── scripts/
│   ├── calculadora_dano.gd
│   ├── sistema_nivel.gd
│   └── jogador.gd
│
├── test/
│   ├── calculadora_dano_test.gd
│   ├── sistema_nivel_test.gd
│   └── jogador_test.gd
│
└── addons/
    └── gdunit4/
```

O GdUnit4 pode ser configurado para localizar os testes na pasta `test`.

Uma classe de teste em GDScript normalmente começa com:

```gdscript
extends GdUnitTestSuite
```

E os métodos de teste devem iniciar com:

```text
test_
```

Exemplo:

```gdscript
func test_deve_calcular_dano() -> void:
    pass
```

---

# 4. Primeiro exemplo – Calculadora de Dano

Vamos começar com uma unidade simples e sem dependências externas.

Crie a pasta:

```text
res://scripts/
```

Dentro dela, crie:

```text
calculadora_dano.gd
```

Adicione o código:

```gdscript
class_name CalculadoraDano

func calcular(ataque: int, defesa: int) -> int:
    var dano := ataque - defesa

    if dano < 0:
        dano = 0

    return dano
```

A regra é:

```text
dano = ataque - defesa
```

Entretanto, o dano nunca pode ser menor que zero.

---

# 5. Criando o primeiro teste

Crie a pasta:

```text
res://test/
```

Crie o arquivo:

```text
calculadora_dano_test.gd
```

Adicione:

```gdscript
extends GdUnitTestSuite

func test_deve_calcular_dano() -> void:
    var calculadora := CalculadoraDano.new()

    var resultado := calculadora.calcular(50, 20)

    assert_int(resultado).is_equal(30)
```

Observe a estrutura:

```text
Preparar → Executar → Verificar
```

Ou, utilizando o padrão **Arrange – Act – Assert**:

```gdscript
func test_deve_calcular_dano() -> void:

    # Arrange
    var calculadora := CalculadoraDano.new()

    # Act
    var resultado := calculadora.calcular(50, 20)

    # Assert
    assert_int(resultado).is_equal(30)
```

---

# 6. Executando o teste

O GdUnit4 permite executar testes diretamente pelo editor.

Dependendo da versão e da configuração do plugin, você pode:

- utilizar o **GdUnit Inspector**;
- executar a suíte pelo menu de contexto do arquivo de teste;
- executar um teste individual pelo editor de scripts;
- executar todos os testes encontrados pelo GdUnit4.

Execute:

```text
test_deve_calcular_dano
```

O resultado esperado é:

```text
PASS
```

---

# 7. Fazendo o teste falhar

Altere temporariamente a implementação:

```gdscript
func calcular(ataque: int, defesa: int) -> int:
    return ataque + defesa
```

Execute novamente o teste.

Para:

```text
ataque = 50
defesa = 20
```

o teste espera:

```text
30
```

mas receberá:

```text
70
```

O teste deverá falhar.

Depois da demonstração, restaure a implementação correta:

```gdscript
func calcular(ataque: int, defesa: int) -> int:
    var dano := ataque - defesa

    if dano < 0:
        dano = 0

    return dano
```

> Um teste não corrige um defeito. Ele evidencia que o comportamento observado não corresponde ao comportamento esperado.

---

# 8. Criando mais casos de teste

Podemos testar diferentes situações:

```gdscript
extends GdUnitTestSuite

func test_ataque_maior_que_defesa_deve_causar_dano() -> void:
    var calculadora := CalculadoraDano.new()

    assert_int(
        calculadora.calcular(50, 20)
    ).is_equal(30)


func test_ataque_igual_defesa_deve_retornar_zero() -> void:
    var calculadora := CalculadoraDano.new()

    assert_int(
        calculadora.calcular(30, 30)
    ).is_equal(0)


func test_defesa_maior_que_ataque_nao_deve_gerar_dano_negativo() -> void:
    var calculadora := CalculadoraDano.new()

    assert_int(
        calculadora.calcular(20, 40)
    ).is_equal(0)
```

Agora temos três comportamentos diferentes sendo exercitados.

---

# 9. Utilizando `before_test()`

Se todos os testes precisam criar o mesmo objeto, podemos evitar repetição utilizando:

```gdscript
before_test()
```

Exemplo:

```gdscript
extends GdUnitTestSuite

var calculadora: CalculadoraDano

func before_test() -> void:
    calculadora = CalculadoraDano.new()


func test_ataque_maior_que_defesa_deve_causar_dano() -> void:
    assert_int(
        calculadora.calcular(50, 20)
    ).is_equal(30)


func test_defesa_maior_que_ataque_deve_retornar_zero() -> void:
    assert_int(
        calculadora.calcular(20, 40)
    ).is_equal(0)
```

O método:

```gdscript
before_test()
```

é executado antes de **cada caso de teste**.

Isso ajuda a manter os testes independentes entre si.

---

# 10. Testes parametrizados

Quando vários casos possuem a mesma lógica de verificação, podemos utilizar um teste parametrizado.

Em vez de:

```gdscript
func test_caso_1():
    ...

func test_caso_2():
    ...

func test_caso_3():
    ...
```

podemos definir os dados separadamente.

Exemplo:

```gdscript
extends GdUnitTestSuite

var calculadora: CalculadoraDano

func before_test() -> void:
    calculadora = CalculadoraDano.new()


func test_calcular_dano(
        ataque: int,
        defesa: int,
        esperado: int,
        _test_parameters := [
            [50, 20, 30],
            [100, 25, 75],
            [30, 30, 0],
            [20, 40, 0],
            [0, 0, 0]
        ]) -> void:

    assert_int(
        calculadora.calcular(ataque, defesa)
    ).is_equal(esperado)
```

O GdUnit4 executará o mesmo teste uma vez para cada conjunto de valores.

A estrutura:

```text
[entrada 1, entrada 2, resultado esperado]
```

é semelhante à ideia de testes parametrizados utilizada por outros frameworks de teste.

---

# 11. Aplicando técnicas de teste ao exemplo

Antes de criar casos de teste, devemos pensar **por que cada caso existe**.

Para:

```gdscript
func calcular(ataque: int, defesa: int) -> int:
    var dano := ataque - defesa

    if dano < 0:
        dano = 0

    return dano
```

podemos identificar duas classes de comportamento:

```text
ataque >= defesa
ataque < defesa
```

Do ponto de vista estrutural, também existe uma decisão:

```gdscript
if dano < 0:
```

Portanto, um conjunto adequado deve fazer essa condição assumir:

```text
Verdadeiro
Falso
```

Exemplos:

```text
ataque = 20, defesa = 40
ataque = 50, defesa = 20
```

Um terceiro caso interessante ocorre na fronteira:

```text
ataque = defesa
```

---

# 12. Segundo exemplo – Sistema de Níveis

Agora vamos trabalhar com várias faixas de comportamento.

Crie:

```text
res://scripts/sistema_nivel.gd
```

Adicione:

```gdscript
class_name SistemaNivel

func determinar_rank(xp: int) -> String:

    if xp < 0:
        return "INVALIDO"
    elif xp < 100:
        return "BRONZE"
    elif xp < 300:
        return "PRATA"
    elif xp < 600:
        return "OURO"
    else:
        return "LENDARIO"
```

As regras são:

| XP | Rank |
|---:|---|
| menor que 0 | `INVALIDO` |
| 0 a 99 | `BRONZE` |
| 100 a 299 | `PRATA` |
| 300 a 599 | `OURO` |
| 600 ou mais | `LENDARIO` |

---

# 13. Testando o Sistema de Níveis

Crie:

```text
res://test/sistema_nivel_test.gd
```

Adicione:

```gdscript
extends GdUnitTestSuite

var sistema: SistemaNivel

func before_test() -> void:
    sistema = SistemaNivel.new()


func test_determinar_rank(
        xp: int,
        esperado: String,
        _test_parameters := [
            [-1, "INVALIDO"],
            [0, "BRONZE"],
            [50, "BRONZE"],
            [99, "BRONZE"],
            [100, "PRATA"],
            [299, "PRATA"],
            [300, "OURO"],
            [599, "OURO"],
            [600, "LENDARIO"],
            [1000, "LENDARIO"]
        ]) -> void:

    assert_str(
        sistema.determinar_rank(xp)
    ).is_equal(esperado)
```

Esse exemplo é especialmente útil para trabalhar:

- particionamento de equivalência;
- análise de valor limite;
- cobertura de ramos.

---

# 14. Analisando os valores-limite

As mudanças de comportamento ocorrem em:

```text
0
100
300
600
```

Podemos testar valores imediatamente próximos às fronteiras:

```text
-1 / 0 / 1

99 / 100 / 101

299 / 300 / 301

599 / 600 / 601
```

Esses valores podem ser acrescentados ao teste parametrizado.

Exemplo:

```gdscript
func test_limites_de_rank(
        xp: int,
        esperado: String,
        _test_parameters := [
            [-1, "INVALIDO"],
            [0, "BRONZE"],
            [1, "BRONZE"],

            [99, "BRONZE"],
            [100, "PRATA"],
            [101, "PRATA"],

            [299, "PRATA"],
            [300, "OURO"],
            [301, "OURO"],

            [599, "OURO"],
            [600, "LENDARIO"],
            [601, "LENDARIO"]
        ]) -> void:

    assert_str(
        sistema.determinar_rank(xp)
    ).is_equal(esperado)
```

---

# 15. Terceiro exemplo – Vida do Jogador

Agora vamos testar um objeto que possui **estado**.

Crie:

```text
res://scripts/jogador.gd
```

Adicione:

```gdscript
class_name Jogador

var vida: int = 100

func receber_dano(dano: int) -> void:

    if dano < 0:
        push_error("Dano inválido")
        return

    vida -= dano

    if vida < 0:
        vida = 0


func esta_vivo() -> bool:
    return vida > 0
```

A classe possui algumas regras importantes:

```text
vida inicial = 100
```

```text
dano válido reduz a vida
```

```text
vida nunca pode ser menor que 0
```

```text
dano negativo é inválido
```

```text
vida = 0 → jogador não está vivo
```

---

# 16. Criando testes para o jogador

Crie:

```text
res://test/jogador_test.gd
```

Adicione:

```gdscript
extends GdUnitTestSuite

var jogador: Jogador

func before_test() -> void:
    jogador = Jogador.new()


func test_jogador_deve_iniciar_com_100_de_vida() -> void:
    assert_int(jogador.vida).is_equal(100)


func test_dano_deve_reduzir_vida() -> void:
    jogador.receber_dano(30)

    assert_int(jogador.vida).is_equal(70)


func test_dano_maior_que_vida_deve_limitar_vida_a_zero() -> void:
    jogador.receber_dano(150)

    assert_int(jogador.vida).is_equal(0)


func test_jogador_com_vida_deve_estar_vivo() -> void:
    assert_bool(
        jogador.esta_vivo()
    ).is_true()


func test_jogador_sem_vida_nao_deve_estar_vivo() -> void:
    jogador.receber_dano(100)

    assert_bool(
        jogador.esta_vivo()
    ).is_false()
```

---

# 17. Testando um erro com `assert_error()`

Em GDScript, situações inválidas normalmente não são tratadas da mesma forma que exceções Java.

No exemplo:

```gdscript
if dano < 0:
    push_error("Dano inválido")
    return
```

o Godot registra um erro utilizando:

```gdscript
push_error()
```

O GdUnit4 oferece uma assertiva específica para esse tipo de situação.

Exemplo:

```gdscript
func test_dano_negativo_deve_reportar_erro() -> void:

    await assert_error(
        func():
            jogador.receber_dano(-10)
    ).is_push_error("Dano inválido")
```

Também é interessante verificar se a vida não foi alterada:

```gdscript
func test_dano_negativo_nao_deve_alterar_vida() -> void:

    await assert_error(
        func():
            jogador.receber_dano(-10)
    ).is_push_error("Dano inválido")

    assert_int(jogador.vida).is_equal(100)
```

---

# 18. Algumas assertivas úteis do GdUnit4

## Inteiros

```gdscript
assert_int(valor).is_equal(10)
```

```gdscript
assert_int(valor).is_greater(0)
```

## Booleanos

```gdscript
assert_bool(valor).is_true()
```

```gdscript
assert_bool(valor).is_false()
```

## Strings

```gdscript
assert_str(valor).is_equal("BRONZE")
```

## Float

```gdscript
assert_float(valor).is_equal(10.5)
```

## Objetos

```gdscript
assert_object(objeto).is_not_null()
```

## Erros do Godot

```gdscript
await assert_error(
    func():
        alguma_operacao()
).is_push_error("Mensagem esperada")
```

Quando possível, prefira assertivas específicas para o tipo que está sendo verificado.

---

# 19. Criando testes pelo editor

Além de criar os arquivos manualmente, o GdUnit4 oferece integração com o editor.

Dependendo da versão instalada, é possível utilizar o menu de contexto sobre uma função e selecionar uma opção semelhante a:

```text
Create Test
```

ou:

```text
Create TestCase
```

O framework pode gerar automaticamente a estrutura inicial da suíte de teste.

Mesmo utilizando a geração automática, é responsabilidade do desenvolvedor definir:

- os cenários;
- os dados de entrada;
- o resultado esperado;
- as técnicas utilizadas para selecionar os casos.

> Automatizar a criação do arquivo de teste não substitui o projeto dos casos de teste.

---

# 20. Unidade ou integração?

É importante diferenciar o escopo dos testes.

Um teste como:

```gdscript
var calculadora := CalculadoraDano.new()
calculadora.calcular(50, 20)
```

é claramente um teste de uma unidade pequena.

Entretanto, quando começamos a testar:

- uma cena completa;
- `CharacterBody2D`;
- física;
- colisões;
- animações;
- entrada de teclado;
- sinais entre vários objetos;

o teste começa a envolver vários elementos do jogo.

Nesse caso, podemos estar nos aproximando de um **teste de integração ou de componente**.

O GdUnit4 possui o **Scene Runner** para testes desse tipo.

Exemplo de carregamento de uma cena:

```gdscript
var runner := scene_runner("res://scenes/jogador.tscn")
```

Esse recurso pode ser utilizado posteriormente para testar interações, entradas e processamento de cenas.

---

# 21. Atividade prática

Escolha uma pequena mecânica de jogo, por exemplo:

```text
Sistema de dano
Sistema de cura
Pontuação
Experiência
Cooldown
Inventário
Consumo de energia
Cálculo de velocidade
Quantidade de munição
Cálculo de crítico
```

Implemente a unidade em GDScript e desenvolva seus testes utilizando GdUnit4.

## Antes de programar os testes

Identifique:

1. quais são os comportamentos esperados;
2. quais são as classes de equivalência;
3. quais são os valores-limite;
4. quais decisões existem no código;
5. quais ramos precisam ser exercitados.

## Depois

Implemente os testes e justifique a escolha de cada cenário.

Procure utilizar:

- pelo menos uma técnica funcional;
- pelo menos um critério estrutural;
- teste parametrizado quando houver vários dados com a mesma lógica de verificação.

---

# 22. Checklist

Antes de considerar sua suíte concluída, verifique:

- [ ] cada teste possui um objetivo claro;
- [ ] os nomes dos testes descrevem o comportamento esperado;
- [ ] os testes são independentes;
- [ ] os valores-limite relevantes foram considerados;
- [ ] diferentes classes de comportamento foram exercitadas;
- [ ] os principais ramos da unidade foram executados;
- [ ] estados inválidos foram considerados;
- [ ] comportamentos de erro foram verificados quando necessário;
- [ ] testes semelhantes foram parametrizados quando isso melhora a legibilidade;
- [ ] a suíte não depende desnecessariamente de cenas, rede, arquivos ou outros recursos externos.

---

# Referências e documentação

- **GdUnit4 – documentação oficial**  
  https://godot-gdunit-labs.github.io/gdUnit4/latest/

- **GdUnit4 – instalação**  
  https://godot-gdunit-labs.github.io/gdUnit4/latest/first_steps/install/

- **GdUnit4 – Test Cases**  
  https://godot-gdunit-labs.github.io/gdUnit4/latest/testing/test-case/

- **GdUnit4 – Assertions**  
  https://godot-gdunit-labs.github.io/gdUnit4/latest/testing/assert/

- **GdUnit4 – Parameterized Tests**  
  https://godot-gdunit-labs.github.io/gdUnit4/latest/advanced_testing/paramerized_tests/

- **GdUnit4 – Error Assertions**  
  https://godot-gdunit-labs.github.io/gdUnit4/latest/testing/assert-error/

- **GdUnit4 – Scene Runner**  
  https://godot-gdunit-labs.github.io/gdUnit4/latest/advanced_testing/sceneRunner/

- **Godot Engine – documentação**  
  https://docs.godotengine.org/
