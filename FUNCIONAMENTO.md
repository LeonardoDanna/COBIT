# Como o sistema funciona

Tudo está em um único arquivo (`index.html`): HTML, CSS e JavaScript (vanilla, sem dependências externas além das fontes do Google Fonts). A inteligência por trás dos casos e das avaliações vem da API `window.claude.complete(prompt)`, disponível no ambiente de Artifacts do Claude.ai.

## 1. Referência de processos COBIT 5

`COBIT_REF` e `PROC_NAMES` listam todos os processos de governança (APO, MEA) e gestão (BAI, DSS) do COBIT 5 com seus códigos e nomes em português. Essa lista é:

- Enviada nos prompts para a IA, para garantir que ela só use códigos/nomes válidos.
- Usada pela função `fullName(codigo)` para exibir "CÓDIGO — Nome completo" na interface.

## 2. Estado do jogo

O objeto `state` guarda todo o progresso do jogador:

```js
{ xp, cases, streak, best, badges, hardAce, usedThemes }
```

- **xp**: experiência acumulada (define a patente).
- **cases**: número de casos resolvidos.
- **streak**: sequência atual de casos com nota ≥ 7.
- **best**: melhor nota já obtida.
- **badges**: lista de IDs de conquistas já desbloqueadas.
- **hardAce**: vira `true` na primeira vez que o jogador tira nota ≥ 8 num caso de dificuldade ≥ 4.
- **usedThemes**: últimos temas de caso (até 8), enviados à IA para evitar repetição.

O estado é salvo/carregado via `window.storage.get/set` sob a chave `"cobit5-game-state"`, com `try/catch` para não quebrar caso a API de armazenamento não esteja disponível (nesse caso o progresso fica só na sessão atual).

## 3. Patentes e dificuldade

`RANKS` define as patentes por faixa de XP, de **Estagiário de TI** (0 XP) até **CIO** (2800 XP). `rankFor(xp)` calcula a patente atual e a próxima, e `refreshHeader()` atualiza a barra de XP no topo.

A dificuldade do próximo caso é calculada por `difficultyLevel()`:

```
nível = min(5, 1 + floor(cases / 2))
```

Ou seja, sobe um nível a cada 2 casos resolvidos, até o máximo de 5.

## 4. Conquistas (badges)

`BADGE_DEFS` define os marcos e a condição de cada um, verificados por `checkBadges()` após cada resultado:

| Badge | Condição |
|---|---|
| 🎓 Primeiro caso | `cases >= 1` |
| 📚 5 casos resolvidos | `cases >= 5` |
| 🧠 10 casos resolvidos | `cases >= 10` |
| 🔥 3 na sequência | `streak >= 3` |
| ⚡ 5 na sequência | `streak >= 5` |
| 🏆 Nota 10 | `best >= 10` |
| 💎 Nota ≥ 8 em caso difícil | `hardAce === true` |

Novas conquistas aparecem como toasts (notificações temporárias no rodapé da tela).

## 5. Fluxo de uma rodada

1. **`renderStart()`** — tela inicial com a tabela de patentes e o botão para iniciar/continuar.
2. **`startCase()`** — mostra um spinner e chama `askClaude(genCasePrompt(nivel))` para gerar um novo caso.
3. **`renderCase()`** — exibe o caso (título, empresa, cenário, dificuldade em "pips") e a grade de processos COBIT 5 para escolha.
4. **`toggleProc(i)`** — controla a seleção (máximo de 3 processos) e chama `syncJustif()`.
5. **`syncJustif()`** — mantém a caixa de texto sincronizada: para cada processo selecionado, cria automaticamente um cabeçalho (`CÓDIGO — Nome:`) preservando o que o usuário já escreveu embaixo de cada um.
6. **`submitAnswer()`** — valida que o usuário escreveu algo além dos cabeçalhos, mostra o spinner e chama `askClaude(evalPrompt(...))` para avaliar.
7. **`applyResult(r)`** — processa a nota, calcula XP ganho, atualiza patente/sequência/badges e renderiza o veredito.

Em caso de erro na geração do caso ou na avaliação, a tela mostra uma mensagem de erro com botão para tentar novamente — no caso da avaliação, a resposta do usuário (`window._lastJust`/`window._lastSel`) não é perdida e é restaurada por `retryRender()`.

## 6. Prompts enviados à IA

### Geração de caso — `genCasePrompt(nivel)`

Pede um estudo de caso empresarial em português, na perspectiva do diretor, com dificuldade `nivel` (1 a 5), 90–150 palavras, e uma lista de 6 a 9 opções de processos (2 a 3 corretos + distratores plausíveis, incluindo confusões clássicas como BAI05 vs BAI06, DSS02 vs DSS03 etc.). Evita repetir temas recentes (`usedThemes`). Resposta esperada em JSON:

```json
{
  "titulo": "...",
  "empresa": "nome e ramo da empresa",
  "tema": "tema resumido em 4 palavras",
  "cenario": "...",
  "processos": [{"codigo": "APO12", "nome": "Gerenciar Riscos"}]
}
```

### Avaliação da resposta — `evalPrompt(chosen, justification)`

Envia o cenário, as opções apresentadas, os processos escolhidos pelo aluno e sua justificativa. Pede uma avaliação processo a processo, os processos ideais, as pistas do cenário (palavras-chave) e uma nota de 0 a 10. Resposta esperada em JSON:

```json
{
  "avaliacoes": [{"codigo": "APO12", "status": "correto|parcial|incorreto", "explicacao": "..."}],
  "ideais": [{"codigo": "...", "nome": "...", "motivo": "..."}],
  "palavras_chave": [{"trecho": "trecho literal do cenário", "processo": "código indicado"}],
  "nota": 8.5,
  "comentario": "feedback geral curto"
}
```

`askClaude(prompt)` chama `window.claude.complete(prompt)`, remove eventuais blocos ```` ```json ```` e extrai o objeto entre o primeiro `{` e o último `}` antes de fazer `JSON.parse`.

## 7. Cálculo de XP e nota

Em `applyResult(r)`:

```
nota         = nota retornada pela IA, normalizada (vírgula → ponto) e limitada entre 0 e 10
mult         = 1 + (nivel - 1) * 0.25        // 1.0 a 2.0 conforme a dificuldade
streakBonus  = streak >= 2 ? 1 + min(streak, 5) * 0.1 : 1   // até 1.5x, baseado na sequência ANTES deste caso
gained       = round(nota * 10 * mult * streakBonus)
```

Depois:

- `cases += 1`, `xp += gained`, `best = max(best, nota)`.
- Se `nota >= 7`, `streak += 1`; caso contrário, `streak = 0`.
- Se `nota >= 8` e `nivel >= 4`, marca `hardAce = true`.
- Verifica novas conquistas e salva o estado.

## 8. Interface

O CSS define um tema "papelada de banca" (tons petróleo, papel e âmbar), com cartões (`.card`), grade responsiva de processos (`.proc-grid`), barra de XP, "carimbos" animados de nota (`.stamp`) e toasts de notificação. Funções auxiliares de UI: `esc()` (escapa HTML para evitar XSS no conteúdo gerado pela IA), `toast()`, `pips()` (bolinhas de dificuldade) e `badgesHTML()`.
