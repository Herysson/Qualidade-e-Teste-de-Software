# Aula Prática 11.1 – Teste de Validação em Jogos Digitais com Godot e GdUnit4

## Objetivos

Nesta aula prática, utilizaremos **Godot 4**, **GDScript** e **GdUnit4** para realizar testes de validação em um pequeno jogo.

Nas aulas anteriores, testamos:

```text
Unidade
↓
comportamentos isolados

Integração
↓
comunicação entre Player, HUD, sinais e cenas
```

Agora o foco muda.

No **teste de validação**, queremos responder:

> **O jogo se comporta da maneira esperada pelo jogador e pelos requisitos definidos?**

Ao final da prática, você deverá ser capaz de:

- diferenciar teste de integração de teste de validação;
- definir requisitos e critérios de aceitação para um jogo;
- transformar requisitos em casos de teste;
- utilizar o `Scene Runner` do GdUnit4 para executar uma cena real;
- simular ações do jogador;
- verificar resultados visíveis na interface;
- executar uma suíte automatizada de validação;
- relacionar automação e regressão;
- compreender por que testes automatizados não substituem completamente testes com jogadores reais.

---

# 1. Onde estamos?

A estratégia de teste estudada na disciplina progride:

```text
Unidade
   ↓
Integração
   ↓
Validação
   ↓
Sistema
```

No teste de integração, perguntamos:

> Os componentes funcionam corretamente quando trabalham juntos?

No teste de validação, a pergunta muda:

> **O jogo implementado atende aos requisitos e produz o comportamento esperado pelo jogador?**

Pressman e Maxim (2021) destacam que, na validação, o foco está nos requisitos, nas ações visíveis ao usuário e nas saídas que ele consegue reconhecer.

Martins (2016) também destaca que casos de validação podem ser definidos a partir dos requisitos, casos de uso e cenários de utilização.

---

# 2. Qual ferramenta utilizaremos?

Continuaremos utilizando:

- **Godot 4**;
- **GDScript**;
- **GdUnit4**;
- **Scene Runner**.

Não é necessário instalar uma nova ferramenta se o GdUnit4 já foi configurado na aula de teste de integração.

O `Scene Runner` permite:

- carregar uma cena;
- processar frames;
- simular teclado e mouse;
- simular ações configuradas no `Input Map`;
- aguardar o processamento das entradas;
- acessar elementos da cena;
- verificar resultados.

## Documentação oficial

GdUnit4:

```text
https://godot-gdunit-labs.github.io/gdUnit4/latest/
```

Scene Runner:

```text
https://godot-gdunit-labs.github.io/gdUnit4/latest/advanced_testing/sceneRunner/
```

Simulação de Actions:

```text
https://godot-gdunit-labs.github.io/gdUnit4/latest/advanced_testing/scene_runner/actions/
```

---

# 3. O exemplo da aula

Construiremos um pequeno jogo chamado:

```text
Desafio de Pontos
```

A regra será simples:

```text
Pressione ESPAÇO
        ↓
ganha 1 ponto
        ↓
chegou a 3 pontos?
    ┌───┴────┐
   não      sim
    ↓         ↓
continua   VOCÊ VENCEU!
```

Também teremos:

```text
Pressione R
↓
reinicia a partida
```

A simplicidade é intencional.

Nosso objetivo principal não é desenvolver um jogo completo, mas compreender como:

```text
REQUISITO
   ↓
AÇÃO DO JOGADOR
   ↓
RESULTADO VISÍVEL
   ↓
TESTE AUTOMATIZADO
```

se relacionam.

---

# 4. Especificação do jogo

Antes de programar, vamos definir o comportamento esperado.

## RF01 – Estado inicial

> Ao iniciar uma partida, o jogo deve apresentar zero pontos e informar que a partida está em andamento.

Critério de aceitação:

```text
DADO que uma nova partida foi iniciada
QUANDO a cena for carregada
ENTÃO deve ser exibido "Pontos: 0"
E deve ser exibido "Jogando".
```

## RF02 – Pontuação

> O jogador deve receber um ponto sempre que executar a ação `pontuar`.

Critério de aceitação:

```text
DADO que a partida está em andamento
QUANDO o jogador executar a ação pontuar
ENTÃO a pontuação deve aumentar em 1
E o novo valor deve ser apresentado na interface.
```

## RF03 – Vitória

> O jogador deve vencer ao atingir 3 pontos.

Critério de aceitação:

```text
DADO que o jogador possui 2 pontos
QUANDO executar novamente a ação pontuar
ENTÃO a pontuação deve chegar a 3
E a interface deve apresentar "VOCÊ VENCEU!".
```

## RF04 – Partida encerrada

> Depois da vitória, novas ações de pontuação não devem alterar o placar.

Critério de aceitação:

```text
DADO que o jogador já venceu
QUANDO executar novamente a ação pontuar
ENTÃO a pontuação deve continuar em 3.
```

## RF05 – Reinício

> O jogador deve poder reiniciar a partida.

Critério de aceitação:

```text
DADO que uma partida está em andamento ou encerrada
QUANDO o jogador executar a ação reiniciar
ENTÃO a pontuação deve voltar para 0
E o status deve voltar para "Jogando".
```

---

# 5. Dos requisitos para os casos de teste

| Caso | Requisito | Ação do jogador | Resultado esperado |
|---|---|---|---|
| CT01 | RF01 | abrir o jogo | `Pontos: 0` e `Jogando` |
| CT02 | RF02 | pressionar Espaço 1 vez | `Pontos: 1` |
| CT03 | RF03 | pressionar Espaço 3 vezes | `Pontos: 3` e `VOCÊ VENCEU!` |
| CT04 | RF04 | pressionar Espaço após vencer | placar permanece em 3 |
| CT05 | RF05 | pressionar R | `Pontos: 0` e `Jogando` |

Observe a rastreabilidade:

```text
RF02
 ↓
CT02
 ↓
ação "pontuar"
 ↓
resultado esperado
 ↓
teste automatizado
```

---

# 6. Estrutura do projeto

Crie um novo projeto ou utilize um projeto já configurado com GdUnit4.

Sugestão:

```text
TesteValidacaoJogo
```

Estrutura:

```text
res://
├── scenes/
│   └── game.tscn
│
├── scripts/
│   └── game.gd
│
└── tests/
    └── game_validation_test.gd
```

---

# 7. Criando a cena

Crie uma nova cena com um nó:

```text
Node
```

Renomeie para:

```text
Game
```

Adicione três nós do tipo `Label`:

```text
Game
├── Titulo
├── Pontos
└── Status
```

Configure inicialmente:

### Titulo

```text
DESAFIO DE PONTOS
```

### Pontos

```text
Pontos: 0
```

### Status

```text
Jogando
```

Organize visualmente os três textos no centro da janela.

Salve a cena como:

```text
res://scenes/game.tscn
```

> **Atenção:** o caminho precisa coincidir exatamente com o caminho utilizado posteriormente em `scene_runner()`.

---

# 8. Configurando as ações do jogador

Acesse:

```text
Project
→ Project Settings
→ Input Map
```

Crie:

```text
pontuar
```

Associe:

```text
Space
```

Depois crie:

```text
reiniciar
```

Associe:

```text
R
```

Teremos:

```text
pontuar   → Espaço
reiniciar → R
```

---

# 9. Criando o script do jogo

Crie:

```text
res://scripts/game.gd
```

Conteúdo:

```gdscript
extends Node

@onready var label_pontos: Label = $Pontos
@onready var label_status: Label = $Status

var pontos: int = 0
var terminou: bool = false


func _ready() -> void:
    iniciar_partida()


func _input(event: InputEvent) -> void:

    if event.is_action_pressed("reiniciar"):
        iniciar_partida()
        return

    if event.is_action_pressed("pontuar"):
        adicionar_ponto()


func adicionar_ponto() -> void:

    if terminou:
        return

    pontos += 1
    atualizar_interface()

    if pontos >= 3:
        terminou = true
        label_status.text = "VOCÊ VENCEU!"


func iniciar_partida() -> void:

    pontos = 0
    terminou = false

    label_pontos.text = "Pontos: 0"
    label_status.text = "Jogando"


func atualizar_interface() -> void:
    label_pontos.text = "Pontos: %d" % pontos
```

Associe esse script ao nó:

```text
Game
```

---

# 10. Testando manualmente

Execute:

```text
game.tscn
```

Primeiro observe:

```text
Pontos: 0
Jogando
```

Pressione `Espaço` três vezes. O resultado esperado é:

```text
Pontos: 3
VOCÊ VENCEU!
```

Pressione Espaço novamente. O placar deve permanecer:

```text
Pontos: 3
```

Agora pressione `R`.

Resultado:

```text
Pontos: 0
Jogando
```

---

# 11. O teste manual já é validação?

Ele pode fornecer evidências de validação, porque estamos exercitando o jogo a partir de comportamentos especificados e visíveis ao jogador.

Mas existe um problema: todas essas ações precisariam ser repetidas manualmente após cada alteração.

Vamos automatizar uma parte desse processo.

---

# 12. Criando a suíte de validação

Crie:

```text
res://tests/game_validation_test.gd
```

Conteúdo inicial:

```gdscript
extends GdUnitTestSuite

const GAME_SCENE := "res://scenes/game.tscn"
```

---

# 13. Nosso ponto de vista no teste

Na integração, poderíamos testar diretamente:

```gdscript
game.adicionar_ponto()
```

Mas agora queremos aproximar o teste da perspectiva do jogador.

Em vez disso, simularemos a action:

```text
pontuar
```

que foi associada ao `Espaço`.

Assim:

```text
jogador
 ↓
entrada
 ↓
jogo processa
 ↓
resultado visível
```

---

# 14. O Scene Runner

Carregamos a cena com:

```gdscript
var runner := scene_runner(GAME_SCENE)
```

Acessamos a cena:

```gdscript
var game := runner.scene()
```

Simulamos uma ação completa de pressionar e soltar com:

```gdscript
runner.simulate_action_pressed("pontuar")
```

Depois aguardamos seu processamento:

```gdscript
await runner.await_input_processed()
```

---

# 15. Primeira demonstração automatizada

Vamos criar primeiro um teste para ser **visualizado em sala**.

```gdscript
func test_demonstracao_visual() -> void:

    var runner := scene_runner(GAME_SCENE)

    runner.move_window_to_foreground()

    await runner.simulate_frames(20, 50)

    runner.simulate_action_pressed("pontuar")
    await runner.await_input_processed()

    await runner.simulate_frames(20, 50)

    runner.simulate_action_pressed("pontuar")
    await runner.await_input_processed()

    await runner.simulate_frames(20, 50)

    runner.simulate_action_pressed("pontuar")
    await runner.await_input_processed()

    await runner.simulate_frames(30, 50)
```

O `move_window_to_foreground()` traz a janela da cena para frente. Já `simulate_frames(20, 50)` processa frames com atraso, tornando as etapas perceptíveis visualmente.

> Essas esperas são úteis para demonstração. Na suíte normal, os testes podem ser executados muito mais rapidamente.

---

# 16. Automação não é apenas executar ações

O teste anterior executou ações, mas ainda não verificou resultados.

Para termos um teste completo precisamos de:

```text
AÇÃO
+
RESULTADO ESPERADO
```

Por exemplo:

```text
pressionar Espaço
↓
esperado: Pontos: 1
```

---

# 17. CT01 – Validando o estado inicial

```gdscript
func test_estado_inicial_deve_ser_zero_e_jogando() -> void:

    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    var label_pontos: Label = game.get_node("Pontos")
    var label_status: Label = game.get_node("Status")

    assert_str(label_pontos.text).is_equal("Pontos: 0")
    assert_str(label_status.text).is_equal("Jogando")
```

---

# 18. Por que verificar a interface?

Poderíamos verificar diretamente:

```gdscript
game.pontos
```

Mas o jogador não enxerga essa variável.

Na validação, resultados como:

```gdscript
label_pontos.text
```

ou:

```gdscript
label_status.text
```

são especialmente importantes porque correspondem a saídas observáveis pelo jogador.

---

# 19. CT02 – Uma ação deve gerar um ponto

```gdscript
func test_pontuar_uma_vez_deve_exibir_um_ponto() -> void:

    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()
    var label_pontos: Label = game.get_node("Pontos")

    runner.simulate_action_pressed("pontuar")
    await runner.await_input_processed()

    assert_str(label_pontos.text).is_equal("Pontos: 1")
```

---

# 20. A importância de `await_input_processed()`

Depois de:

```gdscript
runner.simulate_action_pressed("pontuar")
```

aguardamos:

```gdscript
await runner.await_input_processed()
```

A sequência correta fica:

```text
simular entrada
↓
aguardar processamento
↓
verificar resultado
```

---

# 21. CT03 – Três ações devem gerar vitória

```gdscript
func test_tres_pontos_devem_gerar_vitoria() -> void:

    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    var label_pontos: Label = game.get_node("Pontos")
    var label_status: Label = game.get_node("Status")

    runner.simulate_action_pressed("pontuar")
    await runner.await_input_processed()

    runner.simulate_action_pressed("pontuar")
    await runner.await_input_processed()

    runner.simulate_action_pressed("pontuar")
    await runner.await_input_processed()

    assert_str(label_pontos.text).is_equal("Pontos: 3")
    assert_str(label_status.text).is_equal("VOCÊ VENCEU!")
```

---

# 22. Melhorando a legibilidade

Podemos criar um método auxiliar:

```gdscript
func pontuar(runner) -> void:
    runner.simulate_action_pressed("pontuar")
    await runner.await_input_processed()
```

Assim, o teste pode utilizar:

```gdscript
await pontuar(runner)
await pontuar(runner)
await pontuar(runner)
```

---

# 23. CT04 – Depois da vitória, o placar não deve mudar

```gdscript
func test_depois_da_vitoria_nao_deve_pontuar() -> void:

    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    var label_pontos: Label = game.get_node("Pontos")
    var label_status: Label = game.get_node("Status")

    await pontuar(runner)
    await pontuar(runner)
    await pontuar(runner)

    assert_str(label_status.text).is_equal("VOCÊ VENCEU!")

    await pontuar(runner)

    assert_str(label_pontos.text).is_equal("Pontos: 3")
```

---

# 24. CT05 – Reiniciar a partida

```gdscript
func test_reiniciar_deve_restaurar_estado_inicial() -> void:

    var runner := scene_runner(GAME_SCENE)

    await runner.simulate_frames(1)

    var game := runner.scene()

    var label_pontos: Label = game.get_node("Pontos")
    var label_status: Label = game.get_node("Status")

    await pontuar(runner)
    await pontuar(runner)
    await pontuar(runner)

    assert_str(label_status.text).is_equal("VOCÊ VENCEU!")

    runner.simulate_action_pressed("reiniciar")
    await runner.await_input_processed()

    assert_str(label_pontos.text).is_equal("Pontos: 0")
    assert_str(label_status.text).is_equal("Jogando")
```

---

# 25. Suíte completa

```gdscript
extends GdUnitTestSuite

const GAME_SCENE := "res://scenes/game.tscn"


func pontuar(runner) -> void:
    runner.simulate_action_pressed("pontuar")
    await runner.await_input_processed()


func test_estado_inicial_deve_ser_zero_e_jogando() -> void:

    var runner := scene_runner(GAME_SCENE)
    await runner.simulate_frames(1)

    var game := runner.scene()
    var label_pontos: Label = game.get_node("Pontos")
    var label_status: Label = game.get_node("Status")

    assert_str(label_pontos.text).is_equal("Pontos: 0")
    assert_str(label_status.text).is_equal("Jogando")


func test_pontuar_uma_vez_deve_exibir_um_ponto() -> void:

    var runner := scene_runner(GAME_SCENE)
    await runner.simulate_frames(1)

    var game := runner.scene()
    var label_pontos: Label = game.get_node("Pontos")

    await pontuar(runner)

    assert_str(label_pontos.text).is_equal("Pontos: 1")


func test_tres_pontos_devem_gerar_vitoria() -> void:

    var runner := scene_runner(GAME_SCENE)
    await runner.simulate_frames(1)

    var game := runner.scene()
    var label_pontos: Label = game.get_node("Pontos")
    var label_status: Label = game.get_node("Status")

    await pontuar(runner)
    await pontuar(runner)
    await pontuar(runner)

    assert_str(label_pontos.text).is_equal("Pontos: 3")
    assert_str(label_status.text).is_equal("VOCÊ VENCEU!")


func test_depois_da_vitoria_nao_deve_pontuar() -> void:

    var runner := scene_runner(GAME_SCENE)
    await runner.simulate_frames(1)

    var game := runner.scene()
    var label_pontos: Label = game.get_node("Pontos")

    await pontuar(runner)
    await pontuar(runner)
    await pontuar(runner)
    await pontuar(runner)

    assert_str(label_pontos.text).is_equal("Pontos: 3")


func test_reiniciar_deve_restaurar_estado_inicial() -> void:

    var runner := scene_runner(GAME_SCENE)
    await runner.simulate_frames(1)

    var game := runner.scene()
    var label_pontos: Label = game.get_node("Pontos")
    var label_status: Label = game.get_node("Status")

    await pontuar(runner)
    await pontuar(runner)
    await pontuar(runner)

    runner.simulate_action_pressed("reiniciar")
    await runner.await_input_processed()

    assert_str(label_pontos.text).is_equal("Pontos: 0")
    assert_str(label_status.text).is_equal("Jogando")
```

---

# 26. Executando todos os testes

Execute:

```text
game_validation_test.gd
```

Resultado esperado:

```text
✓ test_estado_inicial_deve_ser_zero_e_jogando
✓ test_pontuar_uma_vez_deve_exibir_um_ponto
✓ test_tres_pontos_devem_gerar_vitoria
✓ test_depois_da_vitoria_nao_deve_pontuar
✓ test_reiniciar_deve_restaurar_estado_inicial
```

---

# 27. Matriz de rastreabilidade

| Requisito | Caso | Evidência automatizada | Resultado |
|---|---|---|---|
| RF01 | CT01 | `Pontos: 0` + `Jogando` | PASS/FAIL |
| RF02 | CT02 | `Pontos: 1` | PASS/FAIL |
| RF03 | CT03 | `Pontos: 3` + `VOCÊ VENCEU!` | PASS/FAIL |
| RF04 | CT04 | permanece `Pontos: 3` | PASS/FAIL |
| RF05 | CT05 | `Pontos: 0` + `Jogando` | PASS/FAIL |

---

# 28. Validação e regressão

Imagine que alguém altere a regra para vencer com 5 pontos, mas o requisito continue definindo 3.

Ao reexecutar a suíte, CT03 e CT04 devem revelar a divergência.

Assim, os testes de validação automatizados também podem compor uma suíte de **regressão**.

---

# 29. Automação valida tudo?

Não.

A automação consegue verificar muito bem:

- regras objetivas;
- estados;
- pontuação;
- mensagens;
- respostas a entradas;
- transições;
- critérios de aceitação mensuráveis.

Mas perguntas como estas são diferentes:

> A mensagem de vitória é clara?

> O jogador entende o objetivo?

> O feedback visual é suficiente?

> O ritmo parece adequado?

> O jogo é divertido?

Esses aspectos exigem observação de jogadores reais.

---

# 30. Validação com jogadores

Pressman e Maxim discutem testes com usuários representativos, inclusive alfa e beta, como formas importantes de revelar problemas que aparecem no uso real. Esse tipo de avaliação é especialmente relevante em produtos como jogos.

Por isso, complementaremos a automação com um pequeno **playtest**.

---

# 31. Mini playtest

Trabalhe em dupla.

Um aluno será:

```text
DESENVOLVEDOR / OBSERVADOR
```

O outro:

```text
JOGADOR
```

O jogador não deverá ler o código.

Dê apenas a instrução:

> **Jogue até vencer e depois tente iniciar uma nova partida.**

Não explique os controles além das informações existentes no próprio jogo.

Observe:

- identificou rapidamente o objetivo?
- encontrou os controles?
- percebeu quando ganhou um ponto?
- percebeu claramente a vitória?
- conseguiu descobrir como reiniciar?
- precisou pedir ajuda?
- tentou alguma ação não prevista?

---

# 32. Registro do playtest

| Questão | Resultado |
|---|---|
| O jogador compreendeu o objetivo? | Sim / Não |
| Conseguiu pontuar sem ajuda? | Sim / Não |
| Percebeu claramente o placar? | Sim / Não |
| Identificou a vitória? | Sim / Não |
| Conseguiu reiniciar? | Sim / Não |
| Precisou de ajuda? | Sim / Não |

Registre também:

```text
Problema observado:
____________________________________

Comportamento esperado:
____________________________________

Sugestão de melhoria:
____________________________________
```

---

# 33. Automação × playtest

## Automação

```text
Entrada conhecida
↓
resultado esperado
↓
PASS / FAIL
```

Boa para:

- requisitos objetivos;
- repetição;
- regressão;
- respostas determinísticas.

## Playtest

```text
Jogador real
↓
comportamento real
↓
observação
↓
feedback
```

Boa para investigar:

- compreensão;
- usabilidade;
- jogabilidade;
- experiência;
- comportamentos inesperados.

As abordagens se complementam.

---

# 34. Atividade dos alunos – novo requisito

Adicione:

## RF06 – Penalidade

> O jogador deve perder um ponto ao executar a ação `penalidade`, sem permitir pontuação negativa.

Configure:

```text
penalidade → P
```

Exemplo:

```text
Pontos: 2
↓
P
↓
Pontos: 1
```

Mas:

```text
Pontos: 0
↓
P
↓
Pontos: 0
```

---

# 35. Antes de programar

Defina pelo menos dois casos de teste.

| Caso | Estado inicial | Ação | Resultado esperado |
|---|---|---|---|
| CT06 | | penalidade | |
| CT07 | | penalidade | |

Pense especialmente em:

```text
comportamento normal
```

versus:

```text
condição limite
```

---

# 36. Depois implemente

Somente depois de definir os casos:

1. crie a action `penalidade`;
2. implemente o comportamento;
3. crie os testes automatizados;
4. execute toda a suíte novamente;
5. verifique se os testes anteriores continuam passando;
6. atualize a matriz de rastreabilidade.

---

# 37. Desafio adicional

Adicione:

```text
Tempo restante: 10
```

O jogador possui apenas 10 segundos para atingir 3 pontos.

Ao chegar a zero:

```text
FIM DE JOGO
```

O GdUnit4 permite avançar o processamento da cena com:

```gdscript
await runner.simulate_frames(...)
```

Isso torna possível testar comportamentos dependentes de tempo de forma controlada.

---

# 38. Questões para discussão

1. O que diferencia este teste de validação do teste de integração realizado anteriormente?
2. Por que definimos os requisitos antes de escrever os testes?
3. Qual a função de um critério de aceitação?
4. Por que simulamos a action `pontuar` em vez de chamar diretamente `adicionar_ponto()`?
5. O que `simulate_action_pressed()` representa nesta prática?
6. Por que utilizamos `await_input_processed()`?
7. Por que verificar o texto apresentado no HUD é interessante para validação?
8. Um teste automatizado consegue afirmar que um jogo é divertido? Explique.
9. Qual a vantagem de associar cada caso a um requisito?
10. Como esses testes podem ser reutilizados em regressão?
11. Qual a diferença entre um `FAIL` automatizado e uma sugestão de melhoria obtida no playtest?
12. Por que um jogador real pode encontrar problemas que a suíte automatizada não encontrou?

---

# 39. Checklist

- [ ] O GdUnit4 está instalado e habilitado.
- [ ] `game.tscn` está em `res://scenes/`.
- [ ] `game.gd` está em `res://scripts/`.
- [ ] `game_validation_test.gd` está em `res://tests/`.
- [ ] A action `pontuar` está configurada.
- [ ] A action `reiniciar` está configurada.
- [ ] Espaço executa `pontuar`.
- [ ] R executa `reiniciar`.
- [ ] O estado inicial apresenta `Pontos: 0`.
- [ ] CT01 foi implementado.
- [ ] CT02 foi implementado.
- [ ] CT03 foi implementado.
- [ ] CT04 foi implementado.
- [ ] CT05 foi implementado.
- [ ] A suíte completa foi executada.
- [ ] A matriz de rastreabilidade foi preenchida.
- [ ] O mini playtest foi realizado.
- [ ] Os problemas observados foram registrados.
- [ ] RF06 foi especificado antes da implementação.
- [ ] Os testes de RF06 foram definidos antes da implementação.
- [ ] Toda a suíte foi reexecutada após a alteração.

---

# 40. Resumo

Nesta prática trabalhamos com:

```text
REQUISITO
   ↓
CRITÉRIO DE ACEITAÇÃO
   ↓
AÇÃO DO JOGADOR
   ↓
CENA REAL
   ↓
RESULTADO VISÍVEL
   ↓
ASSERTIVA
   ↓
PASS / FAIL
```

O GdUnit4 automatizou comportamentos objetivos, enquanto o playtest acrescentou observação de um jogador real.

> **Validar um jogo significa buscar evidências de que ele atende aos requisitos e às expectativas observáveis do jogador. A automação pode verificar muitos critérios objetivos, mas aspectos de experiência e jogabilidade também exigem contato com usuários reais.**

---

# Referências

## Bibliografia da disciplina

PRESSMAN, Roger S.; MAXIM, Bruce R. **Engenharia de software: uma abordagem profissional**. 9. ed. Porto Alegre: AMGH, 2021.

MARTINS, Marcos Danilo Chiodi. **Testes de software**. 1. ed. Rio de Janeiro: SESES, 2016.

## GdUnit4

```text
https://godot-gdunit-labs.github.io/gdUnit4/latest/
```

Scene Runner:

```text
https://godot-gdunit-labs.github.io/gdUnit4/latest/advanced_testing/sceneRunner/
```

Simulação de Actions:

```text
https://godot-gdunit-labs.github.io/gdUnit4/latest/advanced_testing/scene_runner/actions/
```

## Godot

```text
https://docs.godotengine.org/en/stable/tutorials/inputs/input_examples.html
```
