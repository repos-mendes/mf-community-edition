# Salto duplo

Projeto pessoal: segunda opção nova no menu **Tweaks**, que deixa o Mario dar um segundo pulo no ar. Desligada por padrão e independente do [modo imortal](modo-imortal.md); dá para usar as duas juntas.

**Como ativar:** Opções → Tweaks → Gameplay → **double jump**.

## Comportamento

- Apertar pulo no ar dá um segundo pulo com a mesma força do pulo normal. Segurar o botão faz ele subir mais, como no primeiro.
- Só existe um pulo extra: apertar de novo no ar não faz nada até o Mario aterrissar.
- O pulo extra é recarregado ao tocar o chão, nadar ou escalar.
- Funciona também ao sair de uma beirada sem pular (o primeiro toque usa o coyote time, se ainda estiver valendo, e o seguinte usa o pulo extra).
- O som do segundo pulo é um pouco mais agudo, para diferenciar do primeiro.

## Como foi feito

| Arquivo | Mudança |
|---|---|
| `engine/objects/players/behaviors/player_physics_behavior.gd` | Em `_movement_y()`, um toque de pulo no ar chama `_double_jump()` quando o tweak está ligado e o pulo extra está disponível; o pulo extra é recarregado no chão, na água ou ao escalar |
| `engine/objects/players/player.gd` | Lê o tweak `double_jump` (e o atualiza quando os tweaks mudam) e guarda `double_jump_ready` |
| `project.godot`, `stages/main_menu/objects/tweaks_selections.tscn` | Declaram o tweak (padrão `false`) e a opção no menu |

## Decisões

- **Reaproveitar o pulo do chão.** `_double_jump()` usa a mesma velocidade, o mesmo sinal `jumped_signal` e a mesma aceleração de pulo segurado que o pulo normal, então o segundo pulo tem a sensação do resto do jogo em vez de uma física nova.
- **Usar o toque, não o botão segurado.** O pulo extra só sai com `jumped` (botão recém-apertado). Se usasse `jumping` (botão segurado), segurar o pulo depois do primeiro gastaria o segundo sem querer.
- **Não vale agachado.** Evita conflito com o tweak `crouch_jumping`.
- **A roupa de sapo não muda.** Ela tem um comportamento de física próprio (`player_physics_frog_behavior.gd`), que já pula de outro jeito.

## Testes

Validado com o Godot 4.7.3-rc1 rodando a fase 1-1 sem interface, com um script que aperta o botão de pulo em momentos fixos e mede a altura máxima:

| Caso | Altura máxima |
|---|---|
| Pulo simples (tweak ligado) | 157 px |
| Pulo + segundo toque no ar (tweak ligado) | 277 px |
| Três toques (tweak ligado) | 277 px, sem terceiro pulo |
| Pulo + segundo toque no ar (tweak desligado) | 157 px, o segundo toque é ignorado |
| Salto duplo de novo depois de aterrissar | 277 px, o pulo extra recarregou |
