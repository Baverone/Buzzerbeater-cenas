# Planeador Nightmare — Plano de Implementação

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Uma página web única (PT-PT) que, a partir dos dados da wiki da PXG, escolhe a rotação de lures com Disk 4, o team perfeito por hunt, regista testes de dano e acompanha a hunt ao vivo.

**Architecture:**
- **Dados.** Um script Python (stdlib) lê a wiki e gera `pxg/data/pxg-data.json`.
- **Lógica.** Módulos ES puros em `pxg/src/`, sem DOM e testados com `node --test`: Disk, efetividade, dano, rotação, team, modo hunt, armazenamento.
- **UI.** Vanilla JS em `pxg/src/ui/`. O `esbuild` junta a UI, a lógica e o JSON num só `pxg/dist/planeador-nightmare.html`, publicado como Artifact.

**Tech Stack:**
- Node 22: `node:test`, `fetch` não usado.
- esbuild como devDependency.
- Python 3 stdlib + pytest.
- Playwright pré-instalado, para smoke tests.

**Spec:** `docs/superpowers/specs/2026-11-25-planeador-nightmare-design.md`

## Global Constraints

- **Texto da interface:** todo em português de Portugal.
- **Nada que jogue pelo jogador.** Sem teclas ou cliques enviados ao jogo, sem leitura de ecrã ou memória. Só informação e cronómetro.
- **Disk:** intervalo `N` = `{1: 8, 2: 6, 3: 4, 4: 3}`. Por defeito Disk 4.
- **Tempo na bola até ficar pronta:** o menor `t` inteiro com `t + floor(t/N) >= R`. O tick conta desde a entrada na bola. Fora da bola, desce 1 s/s.
- **Itens** (editáveis):
  - Attack Elixir: +35% (`mult 1.35`) durante 20 s, recarga 210 s.
  - Nightmare Revive: recarga 300 s.
- **Tempos por defeito:** 1 s por golpe; 2 s de combate depois do último golpe; 10 s de deslocação entre lures (por hunt).
- **Efetividade:** super efetivo 2, efetivo 1,75, normal 1, inefetivo 0,75, muito inefetivo/nulo 0,5.
- **Multiplicadores de dano:**
  - Device ×1,15.
  - Afinação: "faltou" ×1,05 no alvo, "sobrou" ×0,95, "fechou" ×1.
  - Classe não uniforme se `max/min > 1,10` no mesmo cooldown.
- **Estimativa de dano:** `cd × fatorTipo × fatorTier × escala`.
  - Fator de tipo: AOE+Damage = 1; Target+Damage sem AOE = 0,3; sem Damage = 0.
  - `DEFAULT_TIER_FACTOR = {T1A:1.3, T1B:1.25, TM:1.2, T1C:1.15, T1H:1.1, TR:1.0, T2:0.9, T3:0.7}`.
- **Fontes de dano** por precedência: `teste` > `importado` > `semelhança` > `estimado`. A UI mostra sempre a fonte.
- **A página é autónoma:**
  - sem pedidos de rede em runtime e sem scripts externos;
  - `localStorage` sempre dentro de try/catch, e a página funciona sem ele.
- **Contrato do Artifact:**
  - `<title>Planeador Nightmare</title>`;
  - cores como tokens em `:root`, redefinidas em `@media (prefers-color-scheme: dark)` com `:root:not([data-theme="light"])` e em `:root[data-theme="dark"]`;
  - `body` com fundo explícito;
  - margem lateral de 16 px;
  - sem scroll horizontal a 360 px.
- **Desempenho:**
  - `planRotation` em modo UI (`{lures:12, depth:3, beam:12}`) corre em < 1000 ms em Node para 6 Pokémon;
  - a pesquisa de team cede o controlo ao browser (não bloqueia a UI).
- **Commits:** todas as mensagens terminam com
  ```
  Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01MhQDzeyuvRvjFTZw1DZ3ep
  ```
  Ramo `claude/sleepy-feynman-75lqyr`; push com `git push -u origin claude/sleepy-feynman-75lqyr`.

## Review Focus

1. **Team sem nenhum lurador:** o planeador devolve `{error:'sem-lurador'}` e a UI diz "Marca pelo menos um Pokémon como lurador". Não pode pendurar nem dar exceção. Teste na Task 5.
2. **Team fraco demais para o alvo**, mesmo com elixir e revive: o plano sai na mesma, com `insufficient:true`, e a UI avisa "Esta equipa não fecha dano nesta hunt". Teste na Task 5.
3. **Pokémon sem golpes AOE com dano**, ou com movimentos não lidos pelo scraper: fica fora dos candidatos e o teste de dano mostra "Sem golpes de área". Testes nas Tasks 4 e 6.
4. **`localStorage` corrompido, inacessível, ou código de importação inválido:** carrega os valores por defeito, reporta o erro e nunca lança exceção. Teste na Task 7.
5. **Toques fora de ordem ou duplicados no modo hunt:** são ignorados sem estragar o estado. Teste na Task 10.

---

## Estrutura de ficheiros

```
pxg/
  package.json            # "type":"module"; scripts test/build/smoke; devDependency esbuild
  .gitignore              # node_modules, dist
  scraper/parse.py        # funções puras sobre wikitext
  scraper/scrape.py       # descarrega a wiki e escreve o JSON
  scraper/tests/test_parse.py, scraper/tests/fixtures/*.wiki
  data/pxg-data.json      # gerado e versionado
  src/disk.js  src/effectiveness.js  src/damage.js  src/rotation.js
  src/team.js  src/huntmode.js  src/store.js  src/ultralab.js  src/references.js
  src/ui/app.js  src/ui/characters.js  src/ui/hunt.js  src/ui/damagetest.js
  src/ui/team.js  src/ui/huntmode.js  src/ui/style.css
  index.html              # modelo com <div id="app">
  build.mjs               # esbuild → dist/planeador-nightmare.html (JS, CSS e JSON embutidos)
  tests/*.test.mjs, tests/fixtures/pyroar-team.json, tests/ui.smoke.mjs
```

## Tipos partilhados (JSDoc; usados por todas as tasks)

```js
/** Move    = { name:string, cd:number, tags:string[], element:string|null, nightmare:boolean } */
/** Pokemon = { name, tier:'T1A'|'T1B'|'T1C'|'TM'|'TR'|'T1H'|'T2'|'T3', roles:string[] /* 'burst'|'continuo'|'tanque'|'velocista'|'suporte' */, elements:string[], clan:string|null, moves:Move[], url:string } */
/** Hunt    = { title, name, url, difficulty:'Alta'|'Baixa', resistance, clans:string[], mobTypes:string[], damageElement:string[], superEffective:string[], effective:string[], berries:string[], notes:string[], location:string } */
/** Data    = { generatedAt, clans:{[clan]:{elements:string[], members:{name,tier,roles}[]}}, pokemon:{[name]:Pokemon}, hunts:Hunt[], items:{attackElixir:{mult:1.35,duration:20,cooldown:210,url}, revive:{cooldown:300,url}}, parseErrors:string[] } */
/** DamageTest = { pokemon:string, kind:'full'|'no50'|'nightmare'|'skill', move?:string, values:number[], device:boolean, multiplier?:number } */
```

---

### Task 1: Base do projeto e matemática do Disk

**Files:**
- Create: `pxg/package.json`, `pxg/.gitignore`, `pxg/src/disk.js`
- Test: `pxg/tests/disk.test.mjs`

**Interfaces:**
- Produces:
  - `DISK_INTERVAL: {1:8,2:6,3:4,4:3}`
  - `ballRecovered(seconds:number, N:number): number`, que devolve `seconds + floor(seconds/N)`
  - `ballTimeToReady(remaining:number, N:number): number`, inteiro; `remaining` arredonda para cima
  - `remainingAfter(remaining, seconds, inBall:boolean, N): number`, que nunca é < 0
  - `remainingAt(remAtBall:number[], ballSince:number, at:number, N): number[]`
  - `earliestRelease(remAtBall:number[], offsets:number[], ballSince:number, now:number, N): number`

- [ ] **Step 1: Write the failing tests** em `pxg/tests/disk.test.mjs`:
  - `ballTimeToReady`:
    - `(60,3)===45`, `(40,3)===30`, `(30,3)===23`, `(20,3)===15`, `(0,3)===0`, `(59.2,3)===45`;
    - para R em 0..150 e N em [3,4,6,8], igual a uma procura linear do menor t.
  - `ballRecovered(9,3)===12`.
  - `remainingAfter`: `(40,30,true,3)===0`; `(40,10,false,3)===30`.
  - `remainingAt([40,20],100,112,3)` → `[24,4]`.
  - `earliestRelease`:
    - `([40,20],[0,1],100,100,3)===130`;
    - `([40,20],[0,1],100,140,3)===140`;
    - `([40],[10],100,100,3)===123`.
- [ ] **Step 2:** Escrever `package.json` com `"type":"module"` e `"scripts":{"test":"node --test tests/","build":"node build.mjs","smoke":"node tests/ui.smoke.mjs"}`. Correr `cd pxg && npm test` → FAIL (módulo inexistente).
- [ ] **Step 3: Implement `src/disk.js`.**
  - `earliestRelease` = `max(now, ballSince + ballTimeToReady(max_i(remAtBall[i]-offsets[i]), N))`.
  - `remainingAt` usa `ballRecovered(at-ballSince)` desde a entrada na bola.
- [ ] **Step 4:** `cd pxg && npm test` → PASS.
- [ ] **Step 5: Commit** `feat(pxg): base do projeto e matemática do Disk`.

### Task 2: Scraper da wiki e dados

**Files:**
- Create: `pxg/scraper/parse.py`, `pxg/scraper/scrape.py`, `pxg/scraper/tests/test_parse.py`
- Fixtures: copiar de `/tmp/claude-0/-home-user-Buzzerbeater-cenas/bcc5fed2-dc64-5e77-9fc7-9aeaa40a51f3/scratchpad/wiki/` (`p1.txt`→`shiny_rhydon.wiki`, `ramp.txt`→`shiny_rampardos.wiki`, `mbarb.txt`→`mega_barbaracle.wiki`, `ob.txt`→`clan_orebound.wiki`, `nh_tyr.txt`→`hunt_tyranitar.wiki`). Se faltarem, voltar a descarregá-los com `index.php?title=<Página>&action=raw`.
- Generate: `pxg/data/pxg-data.json`, `pxg/tests/fixtures/pyroar-team.json`

**Interfaces:**
- Produces (Python):
  - `strip_markup(s)->str`
  - `parse_moves(wikitext, tier)->list[dict]`
  - `parse_pokemon_page(wikitext)->{'tier','elements','roles','moves'}`
  - `parse_clan_page(wikitext)->{'elements':[..], 'members':[{'name','tier','roles'}]}`
  - `parse_hunt_page(wikitext, title, url)->Hunt`
- Produces (ficheiros):
  - `Data` em `pxg/data/pxg-data.json`;
  - `pyroar-team.json = {pokemon:{6 nomes:Pokemon}, hunt:Hunt}`, com os 6 nomes `Shiny Golem`, `Shiny Hariyama`, `Shiny Donphan`, `Shiny Rampardos`, `Shiny Rhydon (TM)`, `Shiny Pupitar` e a hunt `Nightmare Hunt - Pyroar Male`.

- [ ] **Step 1: Write the failing tests** (`pytest`, valores exatos):
  - **Rhydon:** usa a secção "Movimentos PVE" (não a PvP).
    - 9 golpes.
    - `Fissure` tem cd 50 e o nome sem `'''`; tags ⊇ {AOE, Damage}; elemento `Ground`.
    - `Horn Attack` tem elemento `Normal` (o ícone é `Normal1.png`; usar o 2.º parâmetro `|Normal|`).
    - `Hammer Arm` não tem AOE.
  - **Rampardos:**
    - tier `T1H`;
    - `Shadow Rush`: cd 50, `nightmare=True`;
    - `Rock Wrecker`: `nightmare=False`;
    - `Iron Head`: elemento `Steel`.
  - **Mega Barbaracle:**
    - `Hone Claws` sem `Damage`;
    - `Stone Edge` com tags ⊇ {AOE, Target}.
  - **Clã Orebound:**
    - `elements == ['Ground','Rock']` (ordenados, da secção "Bônus de Clã");
    - `Shiny Pupitar` → tier `T1B`, roles `['burst','continuo']`;
    - `Shiny Rhydon (TM)` → tier `TM`;
    - 6 membros `T1H`.
    - Mapa de secções: `Tier 1A→T1A`, `Tier 1B→T1B`, `Tier 1C→T1C`, `Technical Machine (TM)→TM`, `Technical Records (TR)→TR`, `Tier 1H→T1H`, `Tier 2→T2`, `Tier 3→T3`.
    - Mapa de ícones de função: `BDD→burst`, `OTDD→continuo`, `Tank→tanque`, `Speedster→velocista`, `SupportOT→suporte`.
  - **Hunt Tyranitar:**
    - `difficulty 'Alta'`, `resistance 'Lavender'`, `clans ['Gardestrike']`;
    - `mobTypes ['Rock','Dark']`, `damageElement ['Rock']`;
    - `superEffective ['Fighting']`;
    - `effective ['Water','Grass','Ground','Bug','Steel','Fairy']`;
    - `berries` ⊇ {`Lum Berry`, `Charti Berry`}.
    - Os nomes de elementos normalizam-se para Title Case: `FIRE`→`Fire`.
- [ ] **Step 2:** `python3 -m pytest pxg/scraper -q` → FAIL. Instalar o pytest com `pip install pytest` se faltar.
- [ ] **Step 3: Implement `parse.py`.**
  - `nightmare` = `tier=='T1H'` e o nome começa por `Shadow `.
  - Secção de golpes: "Movimentos PVE" ou "Moveset PVE" (sem distinguir maiúsculas); se não existir, "Movimentos"; corta em "Efetividades".
  - O cd aceita `( 15s )` e `(15s)`.
- [ ] **Step 4:** pytest → PASS.
- [ ] **Step 5: Implement `scrape.py`** (`python3 -I pxg/scraper/scrape.py --out pxg/data/pxg-data.json --fixture-out pxg/tests/fixtures/pyroar-team.json`).
  - `urllib`, 8 em paralelo, 3 tentativas.
  - Lê as páginas de clã `Orebound`, `Malefic` e `Volcanic`, e todos os membros.
  - `EXTRA_POKEMON=['Shiny Hariyama']`, com `clan:null`.
  - Hunts: listar via `api.php?action=query&list=allpages&apprefix=Nightmare Hunt -&aplimit=500` e manter as que têm algum dos 3 clãs.
  - `items` são constantes com os valores das Global Constraints e os URLs da wiki.
  - Os falhanços vão para `parseErrors`; nada é inventado.
- [ ] **Step 6: Run it e verificar:**
  ```
  python3 -c "import json;d=json.load(open('pxg/data/pxg-data.json'));print(len(d['clans']['Orebound']['members']),len(d['hunts']),sum(1 for p in d['pokemon'].values() if p['moves'])/len(d['pokemon']),len(d['parseErrors']))"
  ```
  Esperado: ≥100 membros Orebound, ≥40 hunts, ≥0,95 com golpes. Rever `parseErrors`.
- [ ] **Step 7: Commit** `feat(pxg): scraper da wiki e dados gerados`.

### Task 3: Efetividade

**Files:**
- Create: `pxg/src/effectiveness.js`
- Test: `pxg/tests/effectiveness.test.mjs`

**Interfaces:**
- Produces:
  - `TYPE_CHART` (tabela de tipos padrão de 18 tipos; ataque→defesa: 2 / 0.5 / 0)
  - `typeMultiplier(attack:string, defTypes:string[]): number`
  - `moveMultiplier(element:string|null, hunt:Hunt): number`

- [ ] **Step 1: Tests:**
  - `typeMultiplier`:
    - `('Fighting',['Normal'])===2`;
    - `('Ground',['Flying'])===0.5` (nulo vale 0,5 na NW);
    - `('Grass',['Water','Ground'])===2`;
    - `('Water',['Fire','Normal'])===1.75`;
    - `('Normal',['Rock','Dark'])===0.75`;
    - `('Psychic',['Rock','Dark'])===0.5`.
  - `moveMultiplier` com a hunt Tyranitar do fixture:
    - `Fighting` → 2;
    - `Water` → 1,75 (vem da lista da página);
    - `Fire` → 0,75;
    - `null` → 1.
- [ ] **Step 2:** `npm test` → FAIL.
- [ ] **Step 3: Implement.**
  - Primeiro as listas da hunt.
  - Senão, com dois tipos:
    - ambos fracos → 2;
    - um fraco e um neutro → 1,75;
    - um fraco e um resistente → 1;
    - um resistente e um neutro → 0,75;
    - ambos resistentes ou algum nulo → 0,5.
  - Com um tipo: 2 / 1 / 0,5 (nulo → 0,5).
- [ ] **Step 4:** PASS.
- [ ] **Step 5: Commit** `feat(pxg): efetividades`.

### Task 4: Modelo de dano

**Files:**
- Create: `pxg/src/damage.js`
- Test: `pxg/tests/damage.test.mjs`

**Interfaces:**
- Consumes: tipos `Pokemon`, `Move`, `DamageTest`.
- Produces:
  - `TYPE_FACTOR={aoe:1,target:0.3,none:0}`, `DEFAULT_TIER_FACTOR` (Global Constraints)
  - `isAoeDamage(move):boolean`, `comboMoves(p):Move[]` (AOE+Damage, inclui nightmare), `fiftyMoves(p):Move[]` (de `comboMoves` com cd 50 e sem nightmare)
  - `parseDamageCSV(text):{rows:{pokemon,move:string|null,value:number}[], errors:string[]}`. Separador `;`, `,` ou tab; `move` vazio = combo completo.
  - `buildDamageTable({pokemon:Record<string,Pokemon>, tests:DamageTest[], imported:rows, clanElements:string[], tierFactors?}):DamageTable`
  - `DamageTable = { moveDamage(name, moveName):{value:number, source:'teste'|'importado'|'semelhança'|'estimado'}, nonUniform:string[], scale:number }`. Os valores são neutros, sem device e sem elixir.
  - `classKey(p, move, clanElements):string` = `${tier}|${roles[0]??'-'}|${clã|fora}`
  - `lureTargetDefault({team:string[], luradores:string[], pokemon, table, mult:(move)=>number}):number`
  - `nextCalibration(factor:number, result:'faltou'|'fechou'|'sobrou'):number`

- [ ] **Step 1: Tests** com Pokémon inline:
  - `A`: T1C, burst, Rock; golpes `m50` (50, AOE/Damage), `m40` (40, AOE/Damage), `m20` (20, AOE/Damage), `t15` (15, Target/Damage).
  - `B`: igual a `A`, com `n50`, `n40`.
  - `C`: igual a `A`, com `c40`.
  - `H`: T1H, golpes `Shadow X` (50, nightmare), `r50` (50), `r40` (40).
  - `clanElements ['Rock','Ground']`.

  Asserções:
  1. Testes de `A`: full `[1000,1200]`, no50 `[600]` → `m50=500`, `m40=400`, `m20=200`, fonte `teste`.
  2. Full `[1150]` com `device:true` e sem no50 → 1000 repartido pelo cd entre `m50`, `m40` e `m20`, ou seja `m50≈454.5`.
  3. Herança: com o teste 1, `B.n50`=500 e `B.n40`=400, fonte `semelhança`.
  4. Não uniforme: juntar testes de `B` (full `[2000]`, no50 `[1000]`) → `nonUniform` contém `'T1C|burst|clã'`, e `C.c40` tem fonte `estimado`.
  5. `H`: full `[900]`, no50 `[500]`, nightmare `[1300]` com multiplier 1.3 → `r50=400`, `r40=500`, `Shadow X=1000`.
  6. Sem dados: `A.m50 = 50*1*1.15 = 57.5`, `A.t15 = 15*0.3*1.15 = 5.175`, `scale===1`. Com o teste 1, `scale` = mediana de valor/(cd·fatorTipo·fatorTier) dos golpes testados.
  7. `parseDamageCSV("Shiny Golem;;12000\nShiny Golem;Rock Blast;3000\nlixo")` → 2 linhas, 1 erro. A fonte `importado` aplica-se quando não há teste.
  8. `nextCalibration(1,'faltou')===1.05`, `(1,'sobrou')===0.95`, `(1,'fechou')===1`.
  9. Pokémon sem golpes AOE com dano → `comboMoves(p)` vazio.
  10. `lureTargetDefault` = combo completo (todos os `comboMoves` × `mult`) do lurador mais forte + 2 × mediana dos combos completos dos restantes.
- [ ] **Step 2:** FAIL.
- [ ] **Step 3: Implement.** Derivação a partir dos testes de um Pokémon (média dos valores, ÷1,15 se device, ÷`multiplier` se existir):
  ```
  F = full; G = no50
  se G e um só fiftyMove: valor(fifty) = F − G; restantes (comboMoves sem fifty e sem nightmare) = G × cd/Σcd
  senão: F × cd/Σcd por comboMoves sem nightmare
  nightmare: valor direto. kind 'skill': substitui o valor desse golpe
  herança: pontos (cd, valor) de 'teste'/'importado' por classKey; mesmo cd → média; interpolação linear em cd; fora do intervalo → proporcional (valor/cd do ponto mais próximo)
  ```
  No T1H, "full" já é o combo sem Shadow e "no50" é o combo sem Shadow e sem a outra skill de 50, por isso as regras acima aplicam-se tal e qual.
- [ ] **Step 4:** PASS.
- [ ] **Step 5: Commit** `feat(pxg): modelo de dano com testes, semelhança e estimativa`.

### Task 5: Simulador e otimizador de rotação

**Files:**
- Create: `pxg/src/rotation.js`
- Test: `pxg/tests/rotation.test.mjs`

**Interfaces:**
- Consumes: `disk.js` (Task 1), `moveMultiplier` (Task 3), `DamageTable` e `comboMoves` (Task 4).
- Produces:
  ```js
  /** Ctx = { pokemon, team:string[], luradores:string[], device:string|null, disk:1|2|3|4, table:DamageTable, hunt:Hunt,
              target:number, times:{secPerMove:1, combatAfter:2, travel:10}, items:{elixir:{mult,duration,cooldown}, revive:{cooldown}} } */
  /** State = { t:number, mons:{[name]:{out:boolean, ballSince:number, remAtBall:{[move]:number}}}, elixirReadyAt:number, reviveReadyAt:number, lastLurador:string|null } */
  /** Step = { pokemon:string, moves:string[], elixir?:boolean, revive?:boolean }   // revive: antes deste passo, repõe os cooldowns do Pokémon */
  /** LureResult = { steps:Step[], start, wait, releaseAt:number[], thresholds:{move:string|null, seconds:number}[], end, damage, surplus, insufficient:boolean, state:State } */
  initialState(ctx):State                         // tudo pronto: ballSince = -1e9, remAtBall = 0
  simulateLure(state, steps:Step[], ctx):LureResult|null   // null se elixir/revive não estiverem prontos
  lureDamage(steps, ctx):number                   // Σ moveDamage × moveMultiplier × (device?1.15) × (elixir?mult)
  candidateLures(state, ctx):Step[][]
  nextLure(state, ctx, opts?):LureResult
  planRotation(ctx, opts={lures:12, depth:3, beam:12}):
     {error:'sem-lurador'} | { lures:LureResult[], cycle:{start:number,length:number}|null, lph:number, avgWait:number, insufficient:boolean }
  signature(steps):string                         // para detetar o ciclo
  ```

- [ ] **Step 1: Tests:**
  - **Uma lure simples.** Um Pokémon com golpes de cd 20 e 30, a partir de `initialState`: `start 0`, `wait 0`, `end = 0+2·1+2+10 = 14`. O Pokémon fica na bola desde `t=2`, com `remAtBall` = `{cd20: 18, cd30: 29}` (golpes usados a 0 e a 1).
  - **Repetir logo a mesma lure.** `wait>0`. O `thresholds[0].seconds` é ≤ ao offset do golpe mais lento e o `move` é esse golpe.
  - **Elixir.** `simulateLure` com `elixir:true` devolve `null` quando `elixirReadyAt > start`.
  - **Revive.** Repõe a 0 todos os cooldowns do lurador antes do segundo combo.
  - **Erros e alvo impossível:**
    - `planRotation` com `luradores:[]` → `{error:'sem-lurador'}`;
    - `target: 1e12` → `insufficient:true`, com 12 lures listadas.
  - **Pyroar** (`tests/fixtures/pyroar-team.json`):
    - Configuração: Disk 4, `buildDamageTable` sem testes, `luradores ['Shiny Golem','Shiny Rampardos']`, device `Shiny Donphan`, `target = lureTargetDefault(...)`, tempos por defeito.
    - Esperado: 12 lures; cada lure tem ≥3 passos, ou um passo com `elixir`, ou um com `revive`; `avgWait ≤ 3`; tempo de execução < 1000 ms.
  - **`signature` e ciclo:** sequências que se repetem com período 6 → `cycle.length===6`.
- [ ] **Step 2:** FAIL.
- [ ] **Step 3: Implement.** Libertação com `earliestRelease`: o passo k só sai depois do recolher do passo k−1, e o recolher é a `release + nGolpes·secPerMove`. O `wait` é `releaseAt[0] − state.t`. O `end` é o último recolher + `combatAfter` + `travel`.

  Algoritmo de candidatos e de pesquisa (não determinado pelos testes):
  ```
  para cada lurador L ∈ luradores (≠ state.lastLurador primeiro):
    variantes(L): todos comboMoves; sem cada fiftyMove/nightmare (uma de cada vez); sem ambos
    adicionar 0..3 outros Pokémon por ordem de prontidão; variantes(P): completo | sem fifty | só o golpe de maior cd
    opcional: elixir no último passo (se pronto); revive em L com segunda variante de L (se pronto)
    guardar só conjuntos mínimos com lureDamage ≥ target (não estender um conjunto que já chega)
  se nenhum chega: devolver os 20 de maior dano, marcados insufficient
  limitar a 400 candidatos, ordenados por (fim, sobra)
  pesquisa em feixe: profundidade `depth`, largura `beam`; custo = fim após `depth` lures + 0.001·sobra + 0.5 se repetir o lurador anterior
  comprometer a 1.ª lure da melhor sequência; repetir até `lures`
  ciclo: menor período p (2..6) tal que signature(lure[i]) === signature(lure[i+p]) para as últimas 2p lures
  ```
- [ ] **Step 4:** PASS (incluindo o limite de tempo).
- [ ] **Step 5: Commit** `feat(pxg): simulador e otimizador de rotação`.

### Task 6: Team perfeito

**Files:**
- Create: `pxg/src/team.js`
- Test: `pxg/tests/team.test.mjs`

**Interfaces:**
- Consumes: `planRotation` (Task 5), `buildDamageTable`, `comboMoves`, `lureTargetDefault` e `nextCalibration` (Task 4).
- Produces:
  - `isLuradorDefault(p):boolean` = `tier==='T1H' || roles.includes('tanque')`
  - `aoeDpm(p, ctx):number` = Σ dano×efetividade/cd×60
  - `prefilter(names, ctx):string[]` = os 12 com maior `aoeDpm` ∪ os 4 luradores com maior `aoeDpm`
  - `bestTeams({base, candidates, owned, calibration=1, onProgress, yieldEvery=5}):Promise<{owned:TeamResult|null, possible:TeamResult|null, purchases:{pokemon,gain}[]}>`
  - `TeamResult = {team:string[], device:string, luradores:string[], lph:number, plan, sources:{teste,importado,semelhança,estimado}}`

- [ ] **Step 1: Tests** com um clã sintético de 10 Pokémon:
  - `Forte1..3` T1C com dano alto; `Lura1`, `Lura2` T1H; `Fraco1..4` T3; `SemAOE` só com Target.
  - Asserções:
    - `possible.team` contém `Forte1..3` e ≥1 lurador; tamanho ≤6;
    - o device pertence ao team;
    - `owned` só usa Pokémon de `owned`;
    - `purchases[0]` é um `Forte*` que não está em `owned`, com `gain>0`;
    - `SemAOE` nunca entra;
    - `onProgress` é chamado.
- [ ] **Step 2:** FAIL.
- [ ] **Step 3: Implement.**
  - Avaliação: `planRotation(ctx,{lures:8,depth:1,beam:1}).lph`, com `target = lureTargetDefault × calibration`.
  - Arranque guloso: os 2 melhores luradores + os 4 melhores restantes.
  - Melhoria: trocas 1 a 1 com os pré-filtrados até não melhorar, no máximo 3 passagens.
  - Device: testar em cada membro.
  - Compras: para cada pré-filtrado que não esteja em `owned`, partir do melhor team "tenho" e fazer a pesquisa local com esse Pokémon disponível; ganho = lph − lph(owned). Top 3.
  - A cada `yieldEvery` avaliações: `await new Promise(r=>setTimeout(r))`.
- [ ] **Step 4:** PASS.
- [ ] **Step 5: Commit** `feat(pxg): team perfeito e compras`.

### Task 7: Armazenamento

**Files:**
- Create: `pxg/src/store.js`
- Test: `pxg/tests/store.test.mjs`

**Interfaces:**
- Produces:
  ```js
  /** AppState = { version:1, characters:{id,name,clan,disk,owned:string[],team:string[],device:string|null,luradores:string[]}[],
                   tests:DamageTest[], imported:rows, manualPokemon:Pokemon[],
                   hunts:{[`${charId}|${huntTitle}`]:{travel:number, calibration:number}},
                   settings:{secPerMove:1, combatAfter:2, elixir:{mult:1.35,duration:20,cooldown:210}, revive:{cooldown:300}, tierFactors:{}} } */
  defaultState():AppState
  load(storage=globalThis.localStorage):{state:AppState, error:string|null}   // chave 'pxg-planeador-v1'
  save(state, storage=globalThis.localStorage):boolean
  exportCode(state):string; importCode(code):{ok:true,state}|{ok:false,error:string}
  ```
- [ ] **Step 1: Tests:**
  - armazenamento vazio → `defaultState()`;
  - JSON corrompido → valores por defeito, com `error` preenchido;
  - um armazenamento cujo `getItem` lança exceção → valores por defeito, sem lançar;
  - `save` seguido de `load` devolve o mesmo estado;
  - `exportCode` → `importCode` devolve o mesmo estado, com o nome `'Pokémon é'`;
  - `importCode('lixo')` → `ok:false`;
  - estado sem `settings` → completado com os valores por defeito.
- [ ] **Step 2:** FAIL.
- [ ] **Step 3: Implement.** Código = base64 de `encodeURIComponent(JSON)`; validar `version===1` e os campos obrigatórios.
- [ ] **Step 4:** PASS.
- [ ] **Step 5: Commit** `feat(pxg): armazenamento e exportar/importar`.

### Task 8: UI base, build, bonecos, hunt, plano e Ultra Lab

**Files:**
- Create:
  - `pxg/index.html`, `pxg/build.mjs`
  - `pxg/src/ui/app.js`, `pxg/src/ui/characters.js`, `pxg/src/ui/hunt.js`, `pxg/src/ui/style.css`
  - `pxg/src/ultralab.js`, `pxg/src/references.js`
- Test: `pxg/tests/ui.smoke.mjs`

**Interfaces:**
- Consumes: todas as anteriores.
- Produces:
  - `mount(root:HTMLElement, data:Data, storage)`
  - separadores: **Bonecos · Hunt · Modo hunt · Teste de dano · Team perfeito · Ultra Lab**
  - `ULTRA_LAB` (Alphas #1–#4: mecânicas e esquivas tal como na spec §7.2; fórmula de team; Mecha Device +10%; Nightmare Move não funciona)
  - `REFERENCES['Nightmare Hunt - Pyroar Male'|'... Female']` (rotação Pyroar do Mass Roon, spec §1.2)

- [ ] **Step 1: Write the smoke test** (`tests/ui.smoke.mjs`; Playwright via `createRequire(import.meta.url)('playwright')`):
  1. Fazer o build; abrir `dist/planeador-nightmare.html` com `file://` a 390×844.
  2. Sem scroll horizontal: `scrollWidth ≤ 390`.
  3. Criar o boneco "Teste", Orebound. Marcar como "tenho" e pôr no team os 6 Pokémon do fixture Pyroar, com o Shiny Hariyama adicionado à mão.
  4. Na Hunt, escolher "Pyroar Male" e ver 12 linhas de plano, com o texto "solta quando" ou "solta já".
  5. O painel mostra os super efetivos e a referência do Mass Roon.
  6. O separador Ultra Lab mostra "Alpha Subject #1".
  7. Zero erros na consola.
  8. Repetir a 360 px com `prefers-color-scheme: dark`: sem scroll horizontal e com o fundo do `body` diferente de branco.
- [ ] **Step 2:** `npm i -D esbuild` (versão exata fixada em `package.json`); `npm run smoke` → FAIL.
- [ ] **Step 3: Implement.**
  - `build.mjs`: esbuild `bundle`, `format:'iife'`, `minify`. Embute o CSS e o `data/pxg-data.json` como `<script type="application/json" id="pxg-data">`.
  - **Bonecos:** nome, clã (os 3 clãs), Disk; lista de Pokémon do clã com pesquisa, "tenho", "no team" (máx. 6), lurador (o valor por defeito vem de `isLuradorDefault`) e device; adicionar Pokémon à mão; exportar/importar.
  - **Hunt:**
    - lista filtrada pelo clã e pesquisável;
    - painel da hunt com link para a wiki;
    - tempo de deslocação;
    - tabela do plano: n.º da lure; passos ("Golem: Rock Blast → Stone Edge"); elixir/revive; espera; "solta quando <golpe> marcar N s" ou "solta já";
    - resumo: lures/hora, espera média, ciclo de N lures;
    - avisos: sem lurador, insuficiente, fontes de dano, "±1 s".
  - **Ultra Lab:** painel e ajudante de team, que mapeia os Pokémon "tenho" para os papéis TM Tank (TM+tanque), TM Burst (TM+burst), T1B, T1C finalizador (T1C+burst), T2 finalizador (T2+burst) e Livre.
- [ ] **Step 4:** `npm run build && npm run smoke` → PASS; `npm test` continua a passar.
- [ ] **Step 5: Commit** `feat(pxg): interface base, plano de rotação e Ultra Lab`.

### Task 9: Ecrãs de Teste de dano e Team perfeito

**Files:**
- Create: `pxg/src/ui/damagetest.js`, `pxg/src/ui/team.js`
- Modify: `pxg/src/ui/app.js` (registar os separadores), `pxg/tests/ui.smoke.mjs`

**Interfaces:**
- Consumes: `buildDamageTable`, `parseDamageCSV`, `comboMoves`, `fiftyMoves` (Task 4); `bestTeams` (Task 6); `store` (Task 7).

- [ ] **Step 1: Estender o smoke test:**
  1. No Teste de dano, escolher Shiny Golem → a lista mostra só golpes de área.
  2. Escrever full `12000` (2 valores) e no50 `8000` → a fonte do Golem passa a "teste" na tabela de dano.
  3. A secção "Lista mínima de testes" lista ≥1 Pokémon.
  4. Para um T1H (Shiny Rampardos), aparece o campo do Nightmare Move e uma lista de hunts onde o elemento dele dá 1×.
  5. Colar um CSV com 1 linha válida → 1 linha importada.
  6. No Team perfeito, "Calcular" mostra "Melhor team com os teus Pokémon", "Melhor team possível" e "Compras" em < 20 s.
- [ ] **Step 2:** FAIL.
- [ ] **Step 3: Implement.**
  - **Teste de dano:**
    - Por Pokémon: campos full, no50 (escondido se não houver fiftyMove), nightmare (só T1H, com o seletor de hunt que define `multiplier = moveMultiplier`) e skill a skill (opcional); 1 a 3 valores; checkbox device.
    - "Sem golpes de área" quando `comboMoves` está vazio.
    - **Lista mínima:** uma classe por linha (`classKey` do golpe de maior cd), e o 1.º Pokémon "tenho" ainda sem teste dessa classe, com prioridade às classes que entram no team atual.
    - Classes não uniformes destacadas.
  - **Team perfeito:** botão Calcular com barra de progresso (`onProgress`). Cada resultado mostra team, device, luradores, lures/hora e contagem de fontes; "Usar este team" grava-o no boneco.
- [ ] **Step 4:** `npm run build && npm run smoke` → PASS.
- [ ] **Step 5: Commit** `feat(pxg): teste de dano e team perfeito`.

### Task 10: Modo hunt (lógica e ecrã)

**Files:**
- Create: `pxg/src/huntmode.js`, `pxg/src/ui/huntmode.js`
- Test: `pxg/tests/huntmode.test.mjs`
- Modify: `pxg/tests/ui.smoke.mjs`

**Interfaces:**
- Consumes: `initialState`, `nextLure`, `simulateLure` (Task 5); `nextCalibration` (Task 4).
- Produces:
  ```js
  createSession(ctx, now:number):Session   // Session = {ctx, state, current:LureResult, stepIndex:number, lures:number, startedAt:number, calibration:number}
  recall(session, stepIndex:number, now:number):Session   // só aceita stepIndex === session.stepIndex; release inferido = now − nGolpes·secPerMove
  finishLure(session, result:'faltou'|'fechou'|'sobrou', now:number):Session  // só depois de todos os passos; target ×= nextCalibration; calcula a próxima lure a partir do estado real
  lph(session, now):number
  ```
- [ ] **Step 1: Tests:**
  - `createSession` → `current.steps.length ≥ 1`, `stepIndex 0`.
  - `recall` com o índice 1 primeiro é ignorado (o estado fica igual); `recall(0)` duas vezes conta uma só.
  - `finishLure` antes de todos os passos é ignorado.
  - `finishLure('faltou')` → `calibration 1.05` e a nova `current` calculada com `target×1.05`.
  - 3 lures em 60 s → `lph === 180`.
  - Os cooldowns da sessão seguem os tempos reais: depois de um recolher às 20 s, o `remAtBall` do Pokémon reflete o uso a `20 − nGolpes`.
- [ ] **Step 2:** FAIL.
- [ ] **Step 3: Implement** a lógica e o ecrã:
  - **Passo atual em destaque:** Pokémon, golpes por ordem, elixir/revive e "solta quando … marcar N s", com uma contagem decrescente ao vivo até à hora de soltar (atualizada a cada 250 ms).
  - **Botões grandes (≥56 px):** "Recolhi" (ou "Revive usado") e, no fim, "Faltou dano" / "Fechou" / "Sobrou dano".
  - **Informação de topo:** contador de lures, lures/hora e botão Reiniciar.
  - **Ecrã ligado:** `navigator.wakeLock.request('screen')` dentro de try/catch.
  - **Atalhos opcionais:** `Espaço` = Recolhi; `f`/`c`/`s` = Faltou/Fechou/Sobrou.
  - **Guardar:** grava a calibração por boneco e hunt.
- [ ] **Step 4:** Smoke test: começar o modo hunt na Pyroar Male, tocar "Recolhi" em cada passo e depois "Fechou" → o contador mostra "Lure 2". `npm test && npm run build && npm run smoke` → PASS.
- [ ] **Step 5: Commit** `feat(pxg): modo hunt ao vivo`.

### Task 11: Publicar e verificar

**Files:**
- Modify: só o necessário para cumprir o contrato do Artifact.

- [ ] **Step 1:** Chamar `Artifact` com `action:"quickstart"` e `intent:"other"`; carregar a skill `artifact-design`; confirmar que o `dist/planeador-nightmare.html` cumpre o contrato (Global Constraints). Corrigir o que faltar e repetir `npm run build && npm run smoke`.
- [ ] **Step 2:** Correr tudo: `npm test`, `python3 -m pytest pxg/scraper -q`, `npm run smoke`. Tudo PASS.
- [ ] **Step 3:** Publicar `pxg/dist/planeador-nightmare.html`.
  - `icon:"chart"`.
  - `description`: "Rotações com Disk 4, team perfeito e modo hunt para as hunts Nightmare de Orebound, Malefic e Volcanic."
- [ ] **Step 4:** Abrir o URL publicado em Playwright a 390 px e tirar uma captura, para verificação própria.
- [ ] **Step 5: Commit e push** `chore(pxg): build publicado`.
