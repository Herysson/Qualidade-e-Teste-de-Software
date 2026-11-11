# Aula Prática 10.1 – Teste de Integração em Jogos Digitais com Godot e GdUnit4

## Objetivos

Nesta aula prática, utilizaremos **Godot 4**, **GDScript** e **GdUnit4** para realizar testes de integração em uma cena de jogo.

A proposta é manter o exemplo simples e concentrar a atenção na seguinte pergunta:

> **Quando diferentes componentes do jogo são conectados, eles funcionam corretamente em conjunto?**

Ao final da prática, você deverá ser capaz de:

- diferenciar um teste unitário de um teste de integração em um jogo;
- compreender por que cenas e sinais são pontos importantes de integração no Godot;
- utilizar o **Scene Runner** do GdUnit4;
- carregar uma cena real durante um teste;
- acessar diferentes nós da cena;
- provocar uma alteração em um componente;
- verificar se outros componentes integrados reagiram corretamente;
- executar vários testes automaticamente;
- compreender a ideia de integração incremental e regressão.

---

# 1. Ferramentas utilizadas

Nesta aula utilizaremos:

- **Godot 4**;
- **GDScript**;
- **GdUnit4**;
- **Scene Runner** do GdUnit4.

O GdUnit4 já pode ser utilizado para testes unitários, mas também fornece recursos para testar **cenas completas**, sinais, entradas e interação entre nós.

O **Scene Runner** permite carregar uma cena durante um teste, processar frames e acessar seus nós. A documentação do GdUnit4 apresenta esse recurso como apropriado para testes de integração de cenas.

> Se o GdUnit4 já foi instalado na aula de testes unitários, utilize a mesma instalação.

Caso seja necessário instalar novamente, utilize uma versão do GdUnit4 compatível com a versão do Godot utilizada no laboratório.

## Documentação

GdUnit4:

```text
https://godot-gdunit-labs.github.io/gdUnit4/latest/
```

Scene Runner:

```text
https://github.com/godot-gdunit-labs/gdUnit4/blob/master/documentation/doc/_advanced_testing/sceneRunner.md
```

Godot – Nodes and Scenes:

```text
https://docs.godotengine.org/en/stable/getting_started/step_by_step/nodes_and_scenes.html
```

---

# 2. Qual será o exemplo?

Construiremos uma cena muito pequena contendo:

```text
Game
├── Player
└── HUD
    ├── LabelVida
    └── LabelStatus
```

O objetivo será verificar esta integração:

```text
Player
   ↓
altera a vida
   ↓
emite um sinal
   ↓
HUD recebe o sinal
   ↓
atualiza a interface
```

Também teremos:

```text
Player
   ↓
vida chega a zero
   ↓
emite o sinal "morreu"
   ↓
HUD
   ↓
exibe "GAME OVER"
```

Esse cenário é pequeno, mas envolve componentes diferentes trabalhando juntos.

---

# 3. Por que isso é um teste de integração?

Considere apenas o método:

```gdscript
func receber_dano(valor: int) -> void:
    vida -= valor
```

Podemos testá-lo isoladamente.

Esse seria um **teste unitário**.

Agora considere:

```text
Player recebe dano
        ↓
Player altera sua vida
        ↓
Player emite um sinal
        ↓
Game conecta o sinal ao HUD
        ↓
HUD atualiza LabelVida
```

Nesse caso, queremos verificar se **Player + sinais + Game + HUD** funcionam em conjunto.

Esse é o foco desta prática.

---

# 4. Criando o projeto

Crie um novo projeto no Godot.

Sugestão de nome:

```text
TesteIntegracaoJogo
```

Crie a seguinte estrutura de pastas:

```text
res://
├── scenes/
├── scripts/
└── tests/
```

Ao final teremos algo semelhante a:

```text
res://
├── scenes/
│   └── game.tscn
├── scripts/
│   ├── player.gd
│   ├── hud.gd
│   └── game.gd
└── tests/
    └── game_integration_test.gd
```

---

# 5. Criando o Player

Crie:

```text
res://scripts/player.gd
```

Conteúdo:

```gdscript
class_name Player
extends Node

signal vida_alterada(nova_vida: int)
signal morreu

@export var vida_maxima: int = 100

var vida: int = 100


func _ready() -> void:
    vida = vida_maxima


func receber_dano(dano: int) -> void:
    if dano <= 0:
        return

    vida = max(vida - dano, 0)

    vida_alterada.emit(vida)

    if vida == 0:
        morreu.emit()
```

## O que esse script faz?

O Player possui:

```text
vida_maxima = 100
vida = 100
```

Quando:

```gdscript
receber_dano(30)
```

for executado:

```text
100 - 30 = 70
```

Depois o Player emite:

```gdscript
vida_alterada.emit(vida)
```

informando o novo valor.

Se a vida chegar a zero:

```gdscript
morreu.emit()
```

será emitido.

---

# 6. Criando o HUD

Crie:

```text
res://scripts/hud.gd
```

Conteúdo:

```gdscript
class_name HUD
extends CanvasLayer

@onready var label_vida: Label = $LabelVida
@onready var label_status: Label = $LabelStatus


func atualizar_vida(nova_vida: int) -> void:
    label_vida.text = "Vida: %d" % nova_vida


func mostrar_game_over() -> void:
    label_status.text = "GAME OVER"
```

O HUD possui duas responsabilidades nesta prática:

```text
atualizar a vida apresentada
```

e:

```text
mostrar GAME OVER
```

---

# 7. Criando a cena principal

Crie uma nova cena com um nó:

```text
Node
```

Renomeie para:

```text
Game
```

Adicione como filho:

```text
Node
```

Renomeie para:

```text
Player
```

Associe ao Player:

```text
res://scripts/player.gd
```

Agora adicione ao Game:

```text
CanvasLayer
```

Renomeie para:

```text
HUD
```

Associe:

```text
res://scripts/hud.gd
```

Dentro de `HUD`, adicione dois `Label`:

```text
HUD
├── LabelVida
└── LabelStatus
```

Configure inicialmente:

```text
LabelVida:
Vida: 100
```

```text
LabelStatus:
```

Deixe `LabelStatus` vazio.

A árvore deverá ficar:

```text
Game
├── Player
└── HUD
    ├── LabelVida
    └── LabelStatus
```

Salve como:

```text
res://scenes/game.tscn
```

---

# 8. Integrando Player e HUD

Crie:

```text
res://scripts/game.gd
```

Conteúdo:

```gdscript
extends Node

@onready var player: Player = $Player
@onready var hud: HUD = $HUD


func _ready() -> void:
    player.vida_alterada.connect(hud.atualizar_vida)
    player.morreu.connect(hud.mostrar_game_over)

    hud.atualizar_vida(player.vida)
```

Associe esse script ao nó:

```text
Game
```

---

# 9. O que o Game está fazendo?

Observe:

```gdscript
player.vida_alterada.connect(hud.atualizar_vida)
```

Temos:

```text
Player
   ↓
vida_alterada
   ↓
HUD.atualizar_vida()
```

E:

```gdscript
player.morreu.connect(hud.mostrar_game_over)
```

Temos:

```text
Player
   ↓
morreu
   ↓
HUD.mostrar_game_over()
```

O `Game` funciona como o elemento que conecta esses componentes.

---

# 10. Testando manualmente antes da automação

Neste momento ainda não existe um botão para causar dano.

Para verificar rapidamente o comportamento durante o desenvolvimento, podemos adicionar temporariamente ao `game.gd`:

```gdscript
func _input(event: InputEvent) -> void:
    if event.is_action_pressed("ui_accept"):
        player.receber_dano(10)
```

Execute a cena.

Pressione:

```text
Espaço
```

ou a tecla configurada para:

```text
ui_accept
```

A cada execução:

```text
Vida: 100
Vida: 90
Vida: 80
Vida: 70
...
```

Quando chegar a zero:

```text
GAME OVER
```

Depois de confirmar o funcionamento, você pode remover esse método.

> O teste manual é útil, mas precisaria ser repetido por uma pessoa todas as vezes. A seguir vamos automatizar essa verificação.

---

# 11. O Scene Runner

No teste unitário normalmente criamos ou testamos um objeto isoladamente.

No teste de integração desta aula carregaremos:

```text
game.tscn
```

com:

```gdscript
scene_runner(...)
```

Exemplo:

```gdscript
var runner := scene_runner("res://scenes/game.tscn")
```

O `runner` passa a controlar uma instância real da cena.

Podemos acessar a raiz:

```gdscript
var game := runner.scene()
```

e, a partir dela:

```gdscript
game.get_node("Player")
```

ou:

```gdscript
game.get_node("HUD/LabelVida")
```

Também podemos permitir que a cena processe frames:

```gdscript
await runner.simulate_frames(1)
```

Isso é importante quando queremos que inicializações e interações da cena sejam processadas.

---

# 12. Criando a suíte de integração

Crie:

```text
res://tests/game_integration_test.gd
```

Conteúdo inicial:

```gdscript
extends GdUnitTestSuite

const GAME_SCENE := "res://scenes/game.tscn"
```

Cada função iniciada por:

```text
test_
```

será um teste.

---

# 13. Teste 1 – A cena possui os componentes esperados

Adicione:

```gdscript
func test_game_deve_carregar_player_e_hud() -> void:
    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    assert_object(game.get_node("Player")).is_not_null()
    assert_object(game.get_node("HUD")).is_not_null()
    assert_object(game.get_node("HUD/LabelVida")).is_not_null()
```

## O que estamos verificando?

```text
game.tscn
   ↓
carregou?
   ↓
Player existe?
HUD existe?
LabelVida existe?
```

Esse ainda é um teste de integração bastante simples.

Ele verifica se a cena foi montada com os elementos necessários para a interação seguinte.

---

# 14. Executando o primeiro teste

Abra o painel do GdUnit4.

Execute:

```text
game_integration_test.gd
```

ou utilize a opção de execução disponibilizada pelo GdUnit4 no editor.

O resultado esperado é:

```text
PASS
```

O GdUnit4 também permite executar uma suíte ou várias suítes diretamente pelo editor.

---

# 15. Teste 2 – Dano no Player deve atualizar o HUD

Agora testaremos uma interação real entre componentes.

Adicione:

```gdscript
func test_dano_no_player_deve_atualizar_hud() -> void:
    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    var player: Player = game.get_node("Player")
    var label_vida: Label = game.get_node("HUD/LabelVida")

    player.receber_dano(30)

    await runner.simulate_frames(1)

    assert_int(player.vida).is_equal(70)
    assert_str(label_vida.text).is_equal("Vida: 70")
```

---

# 16. Entendendo o teste

Primeiro carregamos a cena:

```gdscript
var runner := scene_runner(GAME_SCENE)
```

Depois:

```gdscript
await runner.simulate_frames(1)
```

permitimos que a cena seja inicializada.

Acessamos:

```gdscript
var player: Player = game.get_node("Player")
```

e:

```gdscript
var label_vida: Label = game.get_node("HUD/LabelVida")
```

Depois provocamos:

```gdscript
player.receber_dano(30)
```

O esperado é:

```text
Player.vida
100
 ↓ dano 30
70
```

Mas não queremos verificar somente o Player.

Também queremos verificar:

```text
Player
 ↓
sinal vida_alterada
 ↓
HUD
 ↓
LabelVida = "Vida: 70"
```

Por isso existem duas verificações:

```gdscript
assert_int(player.vida).is_equal(70)
```

e:

```gdscript
assert_str(label_vida.text).is_equal("Vida: 70")
```

---

# 17. Por que esse teste é importante?

Imagine que:

```gdscript
receber_dano()
```

esteja correto.

Um teste unitário do Player poderia passar:

```text
vida = 70
PASS
```

Mas alguém remove acidentalmente:

```gdscript
player.vida_alterada.connect(hud.atualizar_vida)
```

Agora:

```text
Player.vida = 70
```

mas:

```text
HUD = "Vida: 100"
```

O Player está correto isoladamente.

O HUD também pode estar correto isoladamente.

O defeito está na:

```text
INTEGRAÇÃO
```

entre eles.

---

# 18. Experimento – provocando um defeito de integração

No arquivo:

```text
game.gd
```

comente temporariamente:

```gdscript
# player.vida_alterada.connect(hud.atualizar_vida)
```

Execute novamente:

```text
test_dano_no_player_deve_atualizar_hud
```

O esperado é:

```text
Player.vida = 70
```

mas:

```text
LabelVida = Vida: 100
```

Portanto:

```text
FAIL
```

Esse é um exemplo claro de um **defeito de integração**.

Depois do experimento, restaure:

```gdscript
player.vida_alterada.connect(hud.atualizar_vida)
```

e execute novamente.

---

# 19. Teste 3 – Vida zero deve mostrar Game Over

Agora testaremos outra integração.

Adicione:

```gdscript
func test_vida_zero_deve_exibir_game_over() -> void:
    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    var player: Player = game.get_node("Player")
    var label_status: Label = game.get_node("HUD/LabelStatus")

    player.receber_dano(100)

    await runner.simulate_frames(1)

    assert_int(player.vida).is_equal(0)
    assert_str(label_status.text).is_equal("GAME OVER")
```

O fluxo verificado é:

```text
Player recebe 100 de dano
        ↓
vida = 0
        ↓
Player emite "morreu"
        ↓
Game conecta o sinal
        ↓
HUD.mostrar_game_over()
        ↓
LabelStatus = "GAME OVER"
```

---

# 20. Teste unitário × teste de integração

Compare.

## Teste unitário

```text
Player
 ↓
receber_dano(30)
 ↓
vida == 70?
```

Verifica principalmente uma unidade.

## Teste de integração

```text
Player
 ↓
receber_dano(30)
 ↓
sinal
 ↓
Game
 ↓
HUD
 ↓
LabelVida == "Vida: 70"?
```

Agora diferentes elementos precisam funcionar em conjunto.

---

# 21. Suíte completa

O arquivo:

```text
game_integration_test.gd
```

deverá possuir:

```gdscript
extends GdUnitTestSuite

const GAME_SCENE := "res://scenes/game.tscn"


func test_game_deve_carregar_player_e_hud() -> void:
    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    assert_object(game.get_node("Player")).is_not_null()
    assert_object(game.get_node("HUD")).is_not_null()
    assert_object(game.get_node("HUD/LabelVida")).is_not_null()


func test_dano_no_player_deve_atualizar_hud() -> void:
    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    var player: Player = game.get_node("Player")
    var label_vida: Label = game.get_node("HUD/LabelVida")

    player.receber_dano(30)

    await runner.simulate_frames(1)

    assert_int(player.vida).is_equal(70)
    assert_str(label_vida.text).is_equal("Vida: 70")


func test_vida_zero_deve_exibir_game_over() -> void:
    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    var player: Player = game.get_node("Player")
    var label_status: Label = game.get_node("HUD/LabelStatus")

    player.receber_dano(100)

    await runner.simulate_frames(1)

    assert_int(player.vida).is_equal(0)
    assert_str(label_status.text).is_equal("GAME OVER")
```

---

# 22. Executando todos os testes

Execute a suíte completa pelo GdUnit4.

O resultado esperado é semelhante a:

```text
game_integration_test.gd

✓ test_game_deve_carregar_player_e_hud
✓ test_dano_no_player_deve_atualizar_hud
✓ test_vida_zero_deve_exibir_game_over
```

Temos agora uma pequena **suíte automatizada de testes de integração**.

---

# 23. O que foi automatizado?

Sem automação:

```text
Executar jogo
 ↓
causar dano
 ↓
observar vida
 ↓
repetir
 ↓
causar dano até morrer
 ↓
observar GAME OVER
```

Com GdUnit4:

```text
Executar suíte
      ↓
carregar cena
      ↓
provocar dano
      ↓
verificar Player
      ↓
verificar HUD
      ↓
provocar morte
      ↓
verificar GAME OVER
      ↓
resultado
```

A automação permite executar novamente os mesmos testes de forma consistente.

---

# 24. Integração incremental

Podemos construir e testar a integração gradualmente.

## Incremento 1

Criamos:

```text
Game
├── Player
└── HUD
```

Testamos:

```text
a cena possui todos os componentes?
```

## Incremento 2

Integramos:

```text
Player
 ↓
vida_alterada
 ↓
HUD
```

Testamos:

```text
dano atualiza o HUD?
```

## Incremento 3

Integramos:

```text
Player
 ↓
morreu
 ↓
HUD
```

Testamos:

```text
vida zero exibe GAME OVER?
```

A cada incremento executamos novamente os testes anteriores.

---

# 25. Introdução à regressão

Suponha que os dois primeiros testes estejam funcionando:

```text
Cena → PASS
Dano + HUD → PASS
```

Depois adicionamos:

```text
Game Over
```

Devemos executar:

```text
Cena → ?
Dano + HUD → ?
Game Over → ?
```

Não testamos somente o comportamento novo.

Também verificamos se o que funcionava anteriormente continua funcionando.

Essa reexecução é a base do **teste de regressão**.

---

# 26. Por que não utilizar Mock nesta primeira prática?

Nos testes unitários, um Mock pode ser útil para substituir uma dependência.

Por exemplo:

```text
Player
 ↓
Mock
```

Nesta prática queremos justamente verificar a integração entre componentes reais:

```text
Player REAL
   +
Game REAL
   +
HUD REAL
```

Substituir o HUD por um Mock reduziria o escopo da integração que desejamos observar.

> Nesta primeira prática, portanto, utilizaremos os componentes reais da cena.

---

# 27. Atividade dos alunos

A partir do exemplo construído, implemente uma nova integração envolvendo **cura do personagem**.

## Requisitos

Adicione ao `Player`:

```gdscript
func curar(valor: int) -> void:
```

O comportamento esperado é:

- a vida deve aumentar;
- a vida nunca deve ultrapassar `vida_maxima`;
- após a cura, o sinal `vida_alterada` deve ser emitido;
- o HUD deve apresentar a nova vida.

Exemplo:

```text
Vida inicial: 100
 ↓
Dano: 40
 ↓
Vida: 60
 ↓
Cura: 20
 ↓
Vida: 80
 ↓
HUD: "Vida: 80"
```

Crie pelo menos dois testes de integração.

Um deles deve verificar uma cura comum.

O outro deve verificar o comportamento próximo da vida máxima.

> Escolha os valores de entrada e os resultados esperados antes de implementar os testes.

---

# 28. Desafio adicional

Para quem terminar a atividade anterior, crie uma nova cena ou componente:

```text
Pocao
```

A ideia é chegar a:

```text
Pocao
 ↓
coletada
 ↓
Player.curar()
 ↓
Player emite vida_alterada
 ↓
HUD atualiza
```

Agora o teste deverá envolver pelo menos:

```text
Pocao + Player + HUD
```

Pergunta:

> Se `Player.curar()` funciona isoladamente, isso garante que coletar uma Poção atualizará corretamente o HUD?

---

# 29. Questões para discussão

1. O que diferencia o teste `receber_dano()` isolado do teste realizado com a cena completa?
2. Por que o `Scene Runner` é útil para testes de integração?
3. O que aconteceria se a conexão entre `vida_alterada` e `atualizar_vida` fosse removida?
4. O Player pode estar correto e o teste de integração falhar? Explique.
5. O HUD pode estar correto e o teste de integração falhar? Explique.
6. Por que processamos pelo menos um frame da cena durante os testes?
7. Qual a vantagem de executar toda a suíte automaticamente?
8. Por que devemos reexecutar os testes anteriores após adicionar uma nova integração?
9. Em que situação um Mock seria útil?
10. Por que optamos por não utilizar Mock nesta primeira prática?

---

# 30. Checklist

Antes de finalizar:

- [ ] O projeto Godot foi criado.
- [ ] O GdUnit4 está instalado e habilitado.
- [ ] `player.gd` foi criado.
- [ ] `hud.gd` foi criado.
- [ ] `game.gd` foi criado.
- [ ] `game.tscn` possui Player e HUD.
- [ ] `LabelVida` existe.
- [ ] `LabelStatus` existe.
- [ ] O sinal `vida_alterada` está conectado.
- [ ] O sinal `morreu` está conectado.
- [ ] A suíte de integração foi criada.
- [ ] O teste de carregamento da cena passa.
- [ ] O teste de dano atualiza Player e HUD.
- [ ] O teste de morte apresenta GAME OVER.
- [ ] Todos os testes foram executados em conjunto.
- [ ] O experimento de defeito de integração foi realizado.
- [ ] Os testes da atividade de cura foram implementados.

---

# 31. Resumo

Nesta prática testamos:

```text
Player
   +
Signals
   +
Game
   +
HUD
```

O ponto central foi perceber que:

> **Testar cada componente isoladamente não é suficiente para garantir que eles funcionarão corretamente quando conectados.**

Em jogos digitais, muitos defeitos aparecem justamente nas interações entre:

- cenas;
- nós;
- sinais;
- sistemas de entrada;
- interface;
- física;
- animações;
- inventário;
- pontuação;
- inimigos;
- salvamento.

O GdUnit4 e o Scene Runner permitem automatizar parte dessas verificações dentro do próprio ambiente Godot.

---

# Referências

## GdUnit4

Documentação:

```text
https://godot-gdunit-labs.github.io/gdUnit4/latest/
```

Repositório:

```text
https://github.com/godot-gdunit-labs/gdUnit4
```

## GdUnit4 – Scene Runner

```text
https://github.com/godot-gdunit-labs/gdUnit4/blob/master/documentation/doc/_advanced_testing/sceneRunner.md
```

## Godot – Nodes and Scenes

```text
https://docs.godotengine.org/en/stable/getting_started/step_by_step/nodes_and_scenes.html
```

## Bibliografia da disciplina

- PRESSMAN, Roger S.; MAXIM, Bruce R. *Engenharia de software: uma abordagem profissional*. 9. ed. Porto Alegre: AMGH, 2021.
- MARTINS, Marcos Danilo Chiodi. *Testes de software*. 1. ed. Rio de Janeiro: SESES, 2016.
