# Modo imortal

Projeto pessoal: uma opção nova no menu **Tweaks** que deixa o Mario imortal. Ela é desligada por padrão, então o jogo original continua igual para quem não ativar.

**Como ativar:** Opções → Tweaks → Gameplay → **immortal mario**.

## Comportamento

| Situação | Com o modo ligado |
|---|---|
| Cair num buraco | Volta ao último chão firme em que pisou e pisca por 2 s |
| Lava | Não causa dano; o Mario afunda nela como se fosse água |
| Inimigos, espinhos, objetos caindo, esmagamento | Não causam dano nem tiram o poder |
| Encostar num inimigo | O inimigo morre, como com a estrela (quem é imune à estrela continua imune) |
| Tempo acabar | Não mata |
| Reiniciar a fase pelo pause / comando `kill` | Continua funcionando, para o jogador nunca ficar preso |

## Como foi feito

O jogo roda sobre a [Thunder Engine](https://github.com/repos-mendes/thunder-engine), incluída como submódulo em `engine/`. Toda morte e todo dano do jogador passam por dois métodos de `Player` (`hurt()` e `die()`). Por isso a mudança fica concentrada neles, e não espalhada pelas dezenas de inimigos e armadilhas que chamam esses métodos.

| Arquivo | Mudança |
|---|---|
| `engine/objects/players/player.gd` | Lê o tweak `immortal_mario`, bloqueia `hurt()`/`die()`, mantém o ataque de contato ligado e guarda a última posição segura |
| `engine/scripts/classes/level/level.gd` | Ao cair abaixo da tela, chama `immortal_respawn()` em vez de `die()` |
| `engine/objects/fluid/lava_area.gd` | Com o modo ligado, a lava se comporta como água |
| `engine/components/pause/restart_selection.gd`, `.../commands/kill.gd` | Passam a tag `immortal_bypass`, que é a única forma de matar o jogador nesse modo |
| `project.godot`, `stages/main_menu/objects/tweaks_selections.tscn` | Declaram o tweak (padrão `false`) e a opção no menu |

## Decisões

- **Tweak em vez de mudança fixa.** Segue o padrão que o projeto já usa para variações de jogabilidade (como o `retro_damage_system`), dá para ligar e desligar sem reiniciar o jogo, e não altera nada para quem não usa.
- **Exceção explícita para mortes intencionais.** Em vez de remover o "reiniciar fase", essas chamadas passam a tag `immortal_bypass`. Sem isso, um jogador preso numa fase não teria saída.
- **Só chão firme conta como ponto de volta.** A posição segura só é gravada sobre `StaticBody2D` ou `TileMapLayer`. Plataformas móveis ou que caem ficariam fora do lugar ao voltar e jogariam o Mario de novo no buraco, em loop.
- **Reaproveitar o ataque da estrela.** Os inimigos já sabem reagir ao ataque de contato da estrela (e alguns já são imunes a ele), então ligar esse ataque de forma permanente mantém as regras do jogo coerentes.

## Testes

Validado com o Godot 4.7.3-rc1 (build Thunder Engine) rodando as fases sem interface:

- fase 1-1: `hurt()` não tira o poder, `die()` não mata, cair abaixo da tela devolve o Mario à última posição segura e o "reiniciar" ainda funciona;
- fase 1-4 (castelo): o Mario colocado dentro da lava continua vivo.
