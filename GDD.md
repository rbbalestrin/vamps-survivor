# MAGIC SURVIVORS

**Game Design Document**
Versão 1.0

---

URI ERECHIM
CIÊNCIA DA COMPUTAÇÃO — DESENVOLVIMENTO DE JOGOS
Autor: Renan Bez
Data: Abril de 2026

---

## Sumário

1. [História](#1-história)
2. [Gameplay](#2-gameplay)
3. [Personagens](#3-personagens)
4. [Controles](#4-controles)
5. [Câmera](#5-câmera)
6. [Inimigos](#6-inimigos)
7. [Interface (UI / HUD)](#7-interface-ui--hud)
8. [Direção Artística](#8-direção-artística)
9. [Tecnologia e Arquitetura](#9-tecnologia-e-arquitetura)
10. [Cronograma](#10-cronograma)
11. [Referências](#11-referências)

---

## 1. História

### 1.1. Premissa

Em um reino esquecido, a **Torre dos Magos Caídos** abriu suas portas após séculos de silêncio. Com ela, uma maldição varreu as terras: criaturas da escuridão — morcegos amaldiçoados, magos-sombra errantes e servos das trevas — saem do subterrâneo em hordas cada vez maiores. O último aprendiz vivo da ordem, portador de uma centelha de magia, é convocado a resistir.

### 1.2. Narrativa / Mundo

O jogador assume o papel de um **jovem mago** que precisa sobreviver a sucessivas noites no entorno da Torre. A cada noite que resiste, mais conhecimento arcano é resgatado — novos feitiços são lembrados, relíquias são encontradas, e o portal da Torre lentamente se enfraquece.

O mundo é narrativamente minimalista: não há NPCs nem diálogos durante o combate. A narrativa se manifesta por:

- Pequenos trechos de texto entre partidas (telas de menu, vitória e derrota).
- Descrição dos feitiços e melhorias desbloqueadas.
- Elementos visuais do cenário (ruínas, iluminação, partículas).

### 1.3. Conclusão / Objetivo Narrativo

A campanha é concluída quando o jogador sobrevive à **Última Noite** (sobrevivência até o fim do timer da arena) e derrota as ondas finais. Cada run bem-sucedida é uma tentativa de fechar o ciclo de invasão. O meta-progresso acumulado entre runs representa o fortalecimento da ordem arcana ao longo do tempo.

---

## 2. Gameplay

### 2.1. Conceito / Visão Geral

**Magic Survivors** é um *bullet-heaven roguelike* 2D em perspectiva top-down, inspirado diretamente em *Vampire Survivors*. O jogador movimenta um mago por uma arena aberta enquanto feitiços são lançados automaticamente, derrotando hordas de inimigos que surgem em número crescente. A cada level-up o jogador escolhe uma melhoria aleatória, construindo uma build única a cada partida. Entre partidas, uma moeda persistente libera upgrades de meta-progressão.

### 2.2. Pilares de Design

1. **Combate passivo, posicionamento ativo** — o jogador não mira nem dispara manualmente. Toda a tensão está em se mover, atrair grupos e fugir de cercos.
2. **Decisões a cada level-up** — ao subir de nível, o jogador escolhe uma entre várias opções aleatórias de upgrade. Runs diferentes = builds diferentes.
3. **Curva exponencial de poder** — o jogador começa fraco e termina a partida destruindo telas inteiras de inimigos em segundos.
4. **Runs curtas com meta-progressão** — uma partida dura ~20 minutos; a moeda acumulada persiste entre runs e permite upgrades permanentes.

### 2.3. Loops de Jogabilidade

A estrutura é dividida em dois ciclos principais: o **Micro Loop** (dentro de uma partida) e o **Macro Loop** (entre partidas).

#### Micro Loop (dentro da partida)

```
mover → feitiços atacam automaticamente →
derrotar inimigos → coletar frascos de experiência (experience vials) →
subir de nível → escolher upgrade (1 de 2 opções) →
sobreviver à próxima onda → ...
```

#### Macro Loop (entre partidas)

```
iniciar run → sobreviver/morrer → tela de fim (vitória ou derrota) →
moeda de meta-progressão acumulada é salva em disco →
gastar moeda no meta-menu para comprar upgrades permanentes →
iniciar nova run mais forte
```

### 2.4. Progressão e Dificuldade

A dificuldade escala **em função do tempo de partida** por meio do `arena_time_manager`, que incrementa o nível interno de dificuldade a cada **5 segundos** decorridos. Cada aumento de dificuldade:

- Reduz o intervalo entre spawns de inimigos (até um teto de 0,7 s de redução).
- Introduz novos tipos de inimigos na tabela de spawn (ex.: o `wizard_enemy` entra na composição quando a dificuldade atinge o nível 6).
- Aumenta a densidade da horda de forma contínua até o fim da partida.

| Marco | Evento |
|---|---|
| Início | Spawn apenas de morcegos/zumbis básicos. |
| Dificuldade 6 (≈30 s de partida) | Entrada do mago-sombra (`wizard_enemy`). |
| Partida avançada | Intervalo de spawn aproxima-se do mínimo; tela fica saturada. |
| Fim do timer | Tela de vitória (`end_screen` com jingle de vitória). |
| Morte antes do timer | Tela de derrota (`end_screen.set_defeat`). |

### 2.5. Armas e Feitiços

O jogador começa toda run com a **Espada Arcana**. Novos feitiços e melhorias são desbloqueados via pool de upgrades a cada level-up.

#### Feitiços e upgrades atualmente no pool

| Upgrade / Feitiço | Efeito | Peso | Status |
|---|---|:---:|---|
| Espada Arcana (base) | Projétil que busca automaticamente o inimigo mais próximo dentro de 150 px do jogador, invocado por cooldown. Disponível desde o início da run. | — | IMPLEMENTADO |
| Machado Orbital (`axe`) | Desbloqueia a habilidade do machado: invoca machados que giram em espiral ao redor do jogador por 3 segundos, aumentando o raio até 100 px. | 10 | IMPLEMENTADO |
| Dano do Machado (`axe_damage`) | +10% de dano do machado por nível. Entra no pool apenas após o machado ser desbloqueado. | 10 | IMPLEMENTADO |
| Cadência da Espada (`sword_rate`) | Reduz o cooldown da espada em 10% por nível. | 10 | IMPLEMENTADO |
| Dano da Espada (`sword_damage`) | +15% de dano da espada por nível. | 10 | IMPLEMENTADO |
| Velocidade do Mago (`player_speed`) | +10% de velocidade de movimento por nível. | 5 | IMPLEMENTADO |

#### Feitiços planejados (expansão futura)

| Feitiço | Tipo | Descrição | Status |
|---|---|---|---|
| Bola de Fogo | Projétil | Busca o inimigo mais próximo e explode em área. | [PLANEJADO] |
| Relâmpago | AoE | Raio cai em inimigo aleatório da tela. | [PLANEJADO] |
| Nova Gélida | AoE | Onda circular que parte do mago e congela inimigos. | [PLANEJADO] |
| Orbe Flutuante | Orbital | Orbes giram em volta do mago causando dano contínuo. | [PLANEJADO] |
| Livro Arcano | Projétil | Livros flutuam e atiram runas em direção cardeal. | [PLANEJADO] |

### 2.6. Itens / Power-ups / Coletáveis

| Item | Efeito | Status |
|---|---|---|
| Frasco de Experiência (`experience_vial`) | Dropado por inimigos derrotados. Cada frasco coletado concede XP e também soma à moeda persistente de meta-progressão. | IMPLEMENTADO |
| Coração | Restaura HP. | [PLANEJADO] |
| Moeda dourada | Moeda bônus adicional. Atualmente a moeda persistente é a própria experiência coletada. | [PLANEJADO] |
| Ímã | Atrai todos os frascos visíveis na tela. | [PLANEJADO] |
| Bomba | Dano massivo em todos inimigos na tela. | [PLANEJADO] |

### 2.7. Condições de Vitória e Derrota

- **Vitória:** sobreviver até o fim do timer da arena (`arena_time_manager`); a `end_screen` toca o jingle de vitória.
- **Derrota:** HP do jogador chega a 0 (`health_component.check_death`); `main.on_player_died` abre a `end_screen` em modo derrota.
- Em qualquer caso, `MetaProgression.save()` persiste o progresso em `user://game.save` antes do retorno ao menu.

---

## 3. Personagens

### 3.1. Protagonista — O Aprendiz (Mago)

Único personagem jogável na versão atual.

| Atributo | Valor / Estado |
|---|---|
| HP Máximo | Definido via `HealthComponent.max_health` (padrão `10`, configurável no nó). |
| Velocidade | Definida pelo `VelocityComponent` do jogador (acelera na direção do input). |
| Dano recebido | 1 ponto por intervalo de dano (`damage_interval_timer`) enquanto houver inimigo em contato. |
| Feitiço inicial | Espada Arcana (auto-alvo no inimigo mais próximo). |
| Escala de upgrades | Ao escolher `player_speed`, a velocidade base é multiplicada por `1 + (qtd × 0.1)`. |
| Animações | `AnimationPlayer` com estados `walk` e `RESET` (idle). |
| Espelhamento | `visuals.scale.x` inverte para apontar para a direção de movimento. |

Código de referência: [scenes/game_object/player/player.gd](scenes/game_object/player/player.gd) + [scenes/component/health_component.gd](scenes/component/health_component.gd).

### 3.2. Personagens Alternativos [PLANEJADO]

A versão inicial traz apenas um mago jogável. A expansão planejada inclui variantes:

| Personagem | Feitiço inicial | Traço único |
|---|---|---|
| Maga das Chamas | Bola de Fogo | +20% dano de fogo. |
| Bruxa da Tempestade | Relâmpago | Cooldown base reduzido em 15%. |
| Necromante | Orbe Flutuante | Inimigos mortos têm chance de virar aliados por 3s. |
| Druida | Nova Gélida | +25 HP máximo, –10% velocidade. |

### 3.3. Progressão do Personagem

Durante uma partida, o personagem sobe de nível ao acumular XP proveniente de frascos de experiência. A regra atual (`experience_manager.gd`):

```
nível inicial = 1
XP alvo inicial = 1
a cada level-up: XP alvo += 5 (TARGET_EXPERIENCE_GROWTH)
```

Cada level-up dispara a `upgrade_screen`, que apresenta **2 opções** de upgrade sorteadas via `WeightedTable`. O jogador escolhe uma e a aplica imediatamente à build.

**Meta-progressão (entre runs):** a moeda `meta_upgrade_currency` cresce com toda experiência coletada e é salva em `user://game.save`. Gastá-la no meta-menu libera `MetaUpgrade`s permanentes.

---

## 4. Controles

Jogo controlado por teclado e mouse (gamepad planejado).

| Ação | Teclado | Mouse |
|---|---|---|
| Mover para a esquerda | A / seta esquerda | — |
| Mover para a direita | D / seta direita | — |
| Mover para cima | W / seta para cima | — |
| Mover para baixo | S / seta para baixo | — |
| Pausar | definida como `pause` no `project.godot` (padrão: ESC) | — |
| Confirmar / selecionar (UI) | Enter | Clique esquerdo |
| Navegar menus | setas | mouse |

**Observação:** todos os feitiços são lançados automaticamente em intervalos fixos. Não existe botão de ataque — o jogador foca exclusivamente em **posicionamento** e **escolha de upgrades**. Implementação do movimento: [scenes/game_object/player/player.gd:get_movement_vector](scenes/game_object/player/player.gd).

---

## 5. Câmera

- **Perspectiva:** top-down 2D ortográfica, usando `Camera2D`.
- **Comportamento:** *smooth follow* do jogador por interpolação suave.
- **Zoom:** fixo em 1.0. [PLANEJADO: zoom dinâmico em eventos-chave como level-up e bosses.]
- **Resolução interna (viewport):** 640 × 360 (pixel art nativo).
- **Resolução da janela:** 1920 × 1080, modo fullscreen por padrão (`window/size/mode=3`).
- **Modo de escalonamento:** `viewport` — pixels escalados inteiros, sem borramento.
- **Limites:** nenhum. A arena é conceitualmente infinita; o jogador é o centro.

Código: [scenes/game_object/game_camera/game_camera.gd](scenes/game_object/game_camera/game_camera.gd).

---

## 6. Inimigos

Todos os inimigos usam uma arquitetura componentizada (`HurtboxComponent` para receber dano, `HitboxComponent` para causar dano, `HealthComponent` para HP, `HitFlashComponent` para flash visual, `DeathComponent` para morte, `VelocityComponent` para movimento, `VialDropComponent` para drop de experiência).

### 6.1. Morcego Amaldiçoado (Basic Enemy)

| Atributo | Valor / Estado |
|---|---|
| HP | Configurado via `HealthComponent` do nó. |
| Dano de contato | Configurado via `HitboxComponent.damage`. |
| Velocidade | Definida pelo `VelocityComponent`. Acelera sempre em direção ao jogador. |
| IA | Perseguição direta (`velocity_component.accelerate_to_player()`). |
| Drop | Frasco de experiência via `VialDropComponent` ao morrer. |

Código: [scenes/game_object/basic_enemy/basic_enemy.gd](scenes/game_object/basic_enemy/basic_enemy.gd).

### 6.2. Mago-Sombra (Wizard Enemy)

Segundo inimigo, introduzido quando a dificuldade da arena atinge nível 6.

| Atributo | Valor / Estado |
|---|---|
| HP / Dano | Configurados via `HealthComponent` / `HitboxComponent`. |
| IA | Possui flag `is_moving`. Enquanto ativa, persegue o jogador; enquanto falsa, desacelera (fica parado para lançar ataque). |
| Drop | Frasco de experiência. |

Código: [scenes/game_object/wizard_enemy/wizard_enemy.gd](scenes/game_object/wizard_enemy/wizard_enemy.gd).

### 6.3. Variantes Planejadas

| Inimigo | Velocidade | Comportamento | Status |
|---|---|---|---|
| Esqueleto Corredor | Rápido, frágil | Cerca o jogador. | [PLANEJADO] |
| Golem de Pedra | Lento, tanque | Absorve dano. | [PLANEJADO] |
| Elite (variante qualquer) | Padrão | Brilha em vermelho; dropa gema de XP maior. | [PLANEJADO] |
| Boss final (Arquimago Caído) | Médio | Múltiplas fases, padrões de ataque. | [PLANEJADO] |

### 6.4. Sistema de Spawn

Implementado em `enemy_manager.gd`:

- Timer dispara em intervalo configurável (`timer.wait_time` base).
- Posição de spawn: círculo ao redor do jogador com **raio de 375 px**, fora do viewport.
- Inclui *raycast check* contra paredes: se o ponto de spawn estiver atrás de obstáculo, o vetor é rotacionado em 90° e o spawn é tentado novamente (até 4 tentativas).
- Tabela de spawn é um `WeightedTable`:
  - Nível inicial: `basic_enemy` com peso 10.
  - Dificuldade 6: adiciona `wizard_enemy` com peso 20.
- Cada aumento de dificuldade reduz o `timer.wait_time` em `(0.1/12) × nível`, com um teto de redução de 0,7 s.

Código: [scenes/manager/enemy_manager.gd](scenes/manager/enemy_manager.gd).

---

## 7. Interface (UI / HUD)

A UI é organizada em diversas cenas dedicadas, todas em [scenes/ui/](scenes/ui/).

### 7.1. Tela Inicial / Menu Principal

- Definida como cena principal no `project.godot` (`run/main_scene="res://scenes/ui/main_menu.tscn"`).
- Botões: iniciar run, abrir meta-menu, opções, sair.
- Usa `screen_transition` (autoload) para fade entre telas.

Código: [scenes/ui/main_menu.gd](scenes/ui/main_menu.gd).

### 7.2. HUD em Jogo

| Elemento | Arquivo |
|---|---|
| Barra de experiência (no topo) | [scenes/ui/experience_bar.gd](scenes/ui/experience_bar.gd) |
| Timer da arena | [scenes/ui/arena_time_ui.gd](scenes/ui/arena_time_ui.gd) |
| Barra de HP sobre o jogador | nó `HealthBar` dentro de `player.tscn` |
| Texto flutuante de dano | [scenes/ui/floating_text.gd](scenes/ui/floating_text.gd) |
| Vinheta (escurecimento nas bordas) | [scenes/ui/vignette.gdshader](scenes/ui/vignette.gdshader) |

### 7.3. Tela de Level-Up (Upgrade Screen)

- Aparece automaticamente quando `experience_manager` emite `level_up`.
- Mostra **2 cartas** (`ability_upgrade_card`) com nome, descrição e ícone de cada upgrade sorteado.
- Jogador escolhe uma; o `upgrade_manager` aplica a mudança e libera variantes no pool (ex.: escolher `axe` desbloqueia `axe_damage`).

Código: [scenes/ui/upgrade_screen.gd](scenes/ui/upgrade_screen.gd), [scenes/ui/ability_upgrade_card.gd](scenes/ui/ability_upgrade_card.gd), [scenes/manager/upgrade_manager.gd](scenes/manager/upgrade_manager.gd).

### 7.4. Tela de Pause

- Acionada pela ação `pause` (definida no `project.godot`).
- Pausa a árvore de cena, exibe botões de retorno e saída.

Código: [scenes/ui/pause_menu.gd](scenes/ui/pause_menu.gd).

### 7.5. Tela de Fim de Partida (End Screen)

- Mostra vitória (`play_jingle`) ou derrota (`set_defeat`) conforme o motivo.
- Chama `MetaProgression.save()` para persistir a moeda da run.
- Retorna ao menu principal.

Código: [scenes/ui/end_screen.gd](scenes/ui/end_screen.gd).

### 7.6. Meta-Menu e Opções

| Tela | Função |
|---|---|
| `meta_menu` | Gasta `meta_upgrade_currency` em `MetaUpgrade` permanentes. |
| `meta_upgrade_card` | Cartão individual de cada meta-upgrade, mostrando preço e quantidade atual. |
| `options_menu` | Configurações de som, janela, etc. |
| `sound_button` | Botão reutilizável com efeito sonoro de hover/click. |

---

## 8. Direção Artística

### 8.1. Estilo Visual

- **Técnica:** pixel art em baixa resolução. Viewport interno de 640×360 escalado para 1920×1080 com `stretch/mode=viewport` (preserva pixels nítidos).
- **Paleta:** cores saturadas com alto contraste contra cenários noturnos. O `boot_splash/bg_color` do projeto já define o tom roxo-vinho característico (`#3F2631`). Feitiços usam tons quentes (amarelo, vermelho, laranja) para destaque.
- **Personagens:** silhuetas legíveis em poucos pixels. Mago com capa e chapéu pontudo, cajado visível. `AnimationPlayer` com estados `walk` e `RESET` (idle).
- **Inimigos:** silhuetas distintas para cada tipo. Morcegos com asas em batida, magos-sombra com capa característica.
- **Cenário:** tilemap de ruínas noturnas em [assets/environment/](assets/environment/).
- **Efeitos:** texto flutuante para números de dano (`floating_text`), `hit_flash_component` aplica flash branco ao atingir inimigo, shader de vinheta escurece as bordas ([scenes/ui/vignette.gdshader](scenes/ui/vignette.gdshader)).
- **Referências:** *Vampire Survivors*, *Brotato*, *20 Minutes Till Dawn*, *Halls of Torment*.

### 8.2. Áudio

Sistema de áudio já em funcionamento:

- **Música:** autoload `MusicPlayer` ([scenes/autoload/music_player.gd](scenes/autoload/music_player.gd)), centralizando a trilha entre menu e gameplay.
- **SFX diegéticos:** componentes `RandomStreamPlayerComponent` e `RandomStreamPlayer2DComponent` tocam variantes aleatórias de som por evento (hit, morte do inimigo, dano no jogador, coleta de frasco).
- **Assets:** em [assets/audio/](assets/audio/).

Roadmap de áudio:

| Elemento | Status |
|---|---|
| Música de menu | IMPLEMENTADO |
| Música de gameplay | IMPLEMENTADO |
| SFX de hit (player e inimigo) | IMPLEMENTADO |
| SFX de coleta de XP | IMPLEMENTADO |
| SFX de level-up (stinger) | [PLANEJADO] |
| Música dedicada para boss | [PLANEJADO] |
| Ajuste individual de volume (música vs. SFX) | [PLANEJADO — via `options_menu`] |

### 8.3. Fontes / Tipografia

- Tema centralizado em [resources/theme/theme.tres](resources/theme/theme.tres).
- **Títulos:** fonte pixel *display* com ar mágico.
- **HUD e menus:** fonte pixel *sans-serif* legível em pequenos tamanhos.
- **Corpo de descrição (upgrade cards):** mesma fonte de HUD, com dois níveis hierárquicos (título do upgrade + descrição).

---

## 9. Tecnologia e Arquitetura

### 9.1. Stack

| Item | Escolha |
|---|---|
| Engine | Godot 4.x (Forward Plus) |
| Linguagem | GDScript |
| Versionamento | Git + GitHub |
| Nome interno do projeto | `Vamps Survivor` (em `project.godot`) |
| Nome comercial | **Magic Survivors** |
| Plataforma-alvo primária | Windows e Linux (desktop) |
| Plataforma-alvo secundária | Web export (HTML5) [PLANEJADO] |
| Resolução base | 640 × 360 |
| Janela padrão | 1920 × 1080 fullscreen (`window/size/mode=3`) |
| Diretório de save | `user://` customizado como `VampsSurvivor` |

### 9.2. Arquitetura do Projeto

O projeto segue uma arquitetura **orientada a componentes** (composição sobre herança). Cada entidade (jogador, inimigo, habilidade) é composta por nós reutilizáveis que cuidam de um aspecto isolado do comportamento.

#### Autoloads (singletons globais)

| Autoload | Responsabilidade |
|---|---|
| `GameEvents` | Barramento de sinais globais (player damaged, ability upgrade added, experience vial collected...). |
| `MusicPlayer` | Toca a trilha sonora de forma contínua entre cenas. |
| `ScreenTransition` | Controla fades entre telas. |
| `MetaProgression` | Persistência em disco da moeda e dos upgrades permanentes. |

#### Componentes reutilizáveis ([scenes/component/](scenes/component/))

| Componente | Função |
|---|---|
| `HealthComponent` | HP máximo, HP atual, sinais `health_changed` e `died`. |
| `HitboxComponent` | Area2D que causa dano configurável. |
| `HurtboxComponent` | Area2D que recebe dano; spawna `floating_text` com o valor. |
| `HitFlashComponent` | Flash visual branco no sprite ao receber dano. |
| `DeathComponent` | Substitui o nó por uma animação de morte e limpa a referência. |
| `VelocityComponent` | Aceleração, desaceleração, movimentação. |
| `VialDropComponent` | Dropa `experience_vial` ao morrer. |
| `RandomStreamPlayerComponent` / `RandomStreamPlayer2DComponent` | Toca um som sorteado de uma lista. |

#### Managers ([scenes/manager/](scenes/manager/))

| Manager | Função |
|---|---|
| `arena_time_manager` | Timer da partida; incrementa a dificuldade a cada 5 segundos. |
| `enemy_manager` | Spawna inimigos em intervalos decrescentes, em círculo de raio 375 ao redor do jogador. |
| `experience_manager` | Agrega XP, controla level-up, cresce o alvo em +5 por nível. |
| `upgrade_manager` | Sorteia upgrades via `WeightedTable`, aplica efeitos, controla limites por upgrade e adiciona novas entradas ao pool. |

#### Estrutura de diretórios

```
scenes/
  ability/
    axe_ability/                 Machado orbital (entidade)
    axe_ability_controller/      Timer que invoca o machado
    sword_ability/               Espada de auto-alvo (entidade)
    sword_ability_controller/    Timer que invoca a espada
  autoload/
    game_events.gd               Barramento global de sinais
    music_player.gd              Trilha contínua
    screen_transition.gd         Fade entre telas
    meta_progression.gd          Save persistente
  component/                     Componentes reutilizáveis (ver tabela acima)
  game_object/
    player/                      Jogador
    basic_enemy/                 Morcego
    wizard_enemy/                Mago-sombra
    experience_vial/             Frasco de XP
    game_camera/                 Câmera follow
  main/                          Cena raiz da partida
  manager/                       Managers de sistema (ver tabela acima)
  ui/                            Todas as telas de interface
assets/
  audio/                         Música e SFX
  environment/                   Tilemap
  ui/                            Sprites de UI
resources/
  theme/                         Tema do Godot
  upgrades/                      Recursos .tres de cada upgrade
```

---

## 10. Cronograma

Planejamento em 4 sprints (aprox. 16 semanas). Status: **Completo**, **Em Progresso**, **Planejado**.

### 10.1. Tabela de tarefas

| Tarefa | Sprint 1 | Sprint 2 | Sprint 3 | Sprint 4 | Status |
|---|:---:|:---:|:---:|:---:|---|
| Escrever o GDD | X | | | | Completo |
| Setup do projeto Godot e repositório | X | | | | Completo |
| Sistema de movimento do jogador | X | | | | Completo |
| Câmera follow | X | | | | Completo |
| Menu principal e transições de tela | X | X | | | Completo |
| Sistema de HP e dano (components) | | X | | | Completo |
| Primeiro inimigo (morcego) com IA de chase | | X | | | Completo |
| Primeiro feitiço (Espada Arcana com auto-alvo) | | X | | | Completo |
| Segundo feitiço (Machado Orbital) | | X | | | Completo |
| Segundo inimigo (mago-sombra) | | X | | | Completo |
| Sistema de XP e level-up | | X | | | Completo |
| Sistema de upgrades com pool ponderado | | X | | | Completo |
| Tela de upgrade (upgrade_screen) | | X | | | Completo |
| Spawner de inimigos com escalonamento | | X | | | Completo |
| Timer da arena e progressão de dificuldade | | X | | | Completo |
| HUD (barra de XP, timer, texto flutuante) | | | X | | Completo |
| Meta-progressão (save file + moeda persistente) | | | X | | Completo |
| Tela de fim (vitória / derrota) | | | X | | Completo |
| Menu de opções e pause | | | X | | Completo |
| Áudio base (música + SFX de hit) | | | X | | Completo |
| Balanceamento de cadência, dano e HP | | | X | X | Em Progresso |
| Expansão de feitiços (Bola de Fogo, Relâmpago, Nova, Orbe, Livro) | | | | X | Planejado |
| Expansão de inimigos (esqueleto, golem, elite) | | | | X | Planejado |
| Boss final (Arquimago Caído) | | | | X | Planejado |
| Personagens alternativos (Maga das Chamas, etc.) | | | | X | Planejado |
| Itens de suporte (coração, ímã, bomba) | | | | X | Planejado |
| Arte final e polimento visual | | | | X | Planejado |
| Música dedicada para boss | | | | X | Planejado |
| Export para Web (HTML5) | | | | X | Planejado |
| Playtesting e ajustes finais | | | | X | Planejado |
| Apresentação do projeto | | | | X | Planejado |

### 10.2. Marcos (milestones)

- **Fim da Sprint 1:** protótipo jogável — jogador anda, câmera acompanha, menu principal carrega a partida.
- **Fim da Sprint 2:** loop principal fechado — HP, feitiços com cadência e dano, inimigos perseguindo, XP, level-up e escolha de upgrade.
- **Fim da Sprint 3:** jogo vertical-slice completo — spawner escalonado, HUD, meta-progressão, telas de fim, áudio e pausa.
- **Fim da Sprint 4:** jogo em estado de entrega — expansão de conteúdo (feitiços, inimigos, boss), balanceamento final, arte polida e apresentação.

---

## 11. Referências

### 11.1. Jogos de referência

- **Vampire Survivors** (Poncle, 2022) — referência primária de gameplay, progressão e feel.
- **Brotato** (Blobfish, 2022) — referência de builds e runs curtas.
- **20 Minutes Till Dawn** (flanne, 2022) — referência de escala e estilo.
- **Halls of Torment** (Chasing Carrots, 2023) — referência de arte pixel e legibilidade em alta densidade de inimigos.

### 11.2. Recursos técnicos

- Documentação oficial do Godot 4: <https://docs.godotengine.org/>
- GDQuest — tutoriais de top-down 2D em Godot.
- Kenney.nl — assets livres para prototipação.

### 11.3. Documentos de apoio

- Exemplos de GDD fornecidos pela disciplina (URI Erechim — Ciência da Computação, Desenvolvimento de Jogos):
  - *Documento final design*.
  - *GDD — Zombie Shooter*.
