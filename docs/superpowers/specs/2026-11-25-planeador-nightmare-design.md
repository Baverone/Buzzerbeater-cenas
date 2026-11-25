# Planeador de Rotações Nightmare (PokeXGames) — Desenho

Data: 2026-11-25 · Estado: rascunho v3 para aprovação

## 1. Intenção e porquê

Quem vai usar isto joga PokeXGames (PXG) com vários bonecos, todos com
**Nightmare Disk 4.0**. O objetivo é caçar nas hunts da Nightmare World "100%
eficiente": nunca ficar parado à espera de cooldowns entre lures. A **página
escolhe a rotação**: que Pokémon soltar em cada lure, que skills usar ou
guardar, onde entram o Elixir de Ataque e o Revive, e em que segundo soltar cada
Pokémon.

### 1.1 Mecânica do Disk (confirmada por quem joga e pela wiki)

- O Disk só atua **com o Pokémon dentro da Pokébola**. Fora da bola o cooldown
  desce ao ritmo normal (1 s por segundo).
- Dentro da bola, além do ritmo normal, recupera **1 s extra a cada N s**:
  Disk 1.0 → N=8, 2.0 → N=6, 3.0 → N=4, **4.0 → N=3**.
- Os helds usados são **X-Attack e X-Critical**, nunca X-Cooldown. Os cooldowns
  base são os da wiki.

### 1.2 Como se caça a sério (vídeos do Mass Roon, Orebound)

Tirado das legendas de três vídeos: "Nightmare Disk 4 e Hunt Pyroar",
"432 mobs/hora – Disk 3" e "Como montar o seu time de Ultra Lab".

- **Uma lure é um combo de 2–3 Pokémon**, não um só.
  - Um Pokémon **lura**: off-tank ou T1H, que puxa os bichos e começa o dano.
  - Os outros **completam o dano**: um burst e um finalizador, normalmente o que
    tem device.
- A rotação é um **ciclo fixo de ~6 lures** que se repete para sempre.
- **Elixir de Ataque** e **Revive** são peças da rotação.
  - O elixir substitui um Pokémon que ainda não voltou do cooldown: a lure do
    elixir tem só dois Pokémon.
  - O revive repõe o kit do Pokémon que lura, que assim combo duas vezes na
    mesma lure. Entra na 6.ª lure.
- **Ciclar a skill de 50 s**: os T1H têm duas skills de 50 s (Nightmare Move e
  outra). Usa-se uma numa lure e guarda-se a outra para a seguinte.
  - Com Disk 3, todos os Pokémon da rotação ciclam a sua skill de 50 s. Por isso
    convém Pokémon com **uma só** skill de 50 s.
  - A skill mais forte do T1H fica guardada para a lure do elixir, porque essa só
    tem dois Pokémon.
- **Soltar antes do cooldown acabar**: o Mass Roon solta o off-tank quando a
  skill mais alta marca **10 s**, e o Donphan ou o Pupitar quando marca **15 s**.
  O resto do cooldown acaba enquanto combo os primeiros golpes.
- Regras dele:
  - "Dano a sobrar é potencial perdido": a lure deve fechar à justa.
  - Mais Pokémon por lure quer dizer mais tempo de cada um na bola, logo mais
    proveito do Disk.
- Exemplo: Pyroar com Disk 4, ~436 mobs/h, ~4 min por ciclo.
  - Equipa: Shiny Golem e Shiny Rampardos (luram alternados), Shiny Hariyama e
    Shiny Rhydon TM (burst que muda de time), Shiny Pupitar (T1B), Shiny Donphan
    (device Critical 8, fecha).
  - Ciclo:
    1. Golem + Hariyama + Donphan
    2. Rampardos + Rhydon TM + Pupitar (guarda a M8)
    3. Golem + M8 do Pupitar + Donphan com elixir
    4. Rampardos + Hariyama + Pupitar + uma skill do Donphan
    5. Golem + Rhydon TM + Donphan
    6. Rampardos → Revive → Rampardos + Pupitar

## 2. Objetivos

1. **Rotação automática.** Escolhe-se **clã → hunt → equipa** e a página propõe
   o ciclo de lures. Para cada lure indica:
   - que Pokémon solta, por ordem, e quais lura;
   - que skills usa e quais guarda;
   - quando usar o Elixir de Ataque e o Revive;
   - **espera antes da lure** (s) e, para cada Pokémon, "**solta quando <skill>
     marcar N s**".
   - Resumo: tempo por ciclo, lures/hora e esperas.
2. **Modo hunt ao vivo.** A página mostra o passo atual e o próximo, e decide a
   lure seguinte a partir do estado real dos cooldowns, não só do plano. Tem
   botões grandes para usar no telemóvel ao lado do PC.
3. **Afinação do dano.** No fim de cada lure tocas em **faltou dano / fechou /
   sobrou dano**, e a página ajusta a estimativa de dano dessa hunt e refaz a
   rotação.
4. **Painel da hunt** tirado da wiki:
   - elemento do dano dos bichos;
   - elementos super efetivos e efetivos;
   - berries recomendadas;
   - boss, angry e shiny;
   - resistência e caminho;
   - link para a wiki.
5. **Cobertura:** hunts indicadas para Orebound (27), Malefic (6) e Volcanic (20)
   segundo a wiki, pela prioridade Orebound → Malefic → Volcanic.
6. **Ultra Lab**:
   - painel com as mecânicas dos 4 Alpha Subject;
   - ajudante de equipa com a fórmula do Mass Roon: TM Tank, TM Burst, T1B com
     device, T1C finalizador, T2 finalizador e 1 lugar livre, marcando quais dos
     teus Pokémon encaixam onde;
   - lembretes: Mecha Device obrigatório; Nightmare Move não funciona lá.
7. Equipas guardadas **por boneco**.
8. **Team perfeito por hunt**, escolhido entre **todos** os Pokémon do clã do
   boneco, incluindo os megas novos (Mega Aerodactyl, Mega Excadrill, Mega
   Barbaracle, …), e não só entre os que já tem. A página mostra três coisas:
   - o melhor team só com os Pokémon que o boneco **tem**;
   - o melhor team **possível** no clã;
   - **o que vale a pena comprar**: que Pokémon, se adicionado, mais sobe as
     lures/hora nessa hunt.
9. **Teste de dano**: um ecrã para registar o dano real de cada Pokémon, medido
   no jogo num alvo neutro. Esses números substituem a estimativa (ver 7.3).

## 3. Não-objetivos (fica para depois ou fora)

- **Rotação automática da Ultra Lab.** É uma hunt em dupla com bosses, e o Mass
  Roon não mostrou a rotação. Fica o painel e o ajudante de equipa.
- **Mini-jogo de treino dos bosses da Ultra Lab**: projeto seguinte.
- Posições das lures no mapa. A wiki não as tem.
- O 4.º vídeo (runas / Pokélog): não tem legendas e o YouTube bloqueia o download
  do vídeo neste ambiente.
- Hunts de outros clãs.

## 4. Anti-objetivos

- **Nada que jogue por quem joga**: sem teclas ou cliques enviados ao jogo, sem
  ler memória ou ecrã, sem macros. Isso dá ban. É informação e cronómetro,
  operado à mão.
- **Fingir que sabe o dano exato.** O dano é uma estimativa que se afina; a
  página tem de o dizer.
- Assumir uma equipa que não existe. A equipa é sempre escolhida na página.

## 5. Restrições e dados conhecidos

- Dados da wiki oficial (`wiki.pokexgames.com`, PT), já permitida na rede. Cada
  dado mostra a página de origem.
- **Attack Elixir** (wiki): +35% de ataque durante 20 s, recarga de 3,5 min.
  - O Mass Roon diz que a lure do elixir rende "70%" de uma lure de três
    Pokémon.
  - Valores editáveis.
- **Nightmare Revive** (wiki): reviver um Pokémon, recarga de **5 min**.
  - Choca com o ciclo de ~4 min do Mass Roon: ou o ciclo real é mais longo, ou
    a recarga é outra.
  - Valor editável; a página respeita-o.
- O jogo tem o foco do teclado. O modo hunt tem de funcionar com **toques no
  telemóvel/tablet**. Os atalhos de teclado são um extra.
- Entrega como página web única (HTML + JS + dados embutidos), com link privado
  para abrir no PC ou no telemóvel. Código e script de recolha no repositório,
  ramo `claude/sleepy-feynman-75lqyr`.
- Interface em português de Portugal.

## 6. Abordagens consideradas

### 6.1 Forma de entrega

- **A. Planeador interativo (escolhida)**: funciona para qualquer equipa e
  recalcula sozinho.
- **B. Guia escrito com equipa "padrão"**: rejeitada. Supõe equipas e daria 50+
  hunts de texto quase repetido.
- **C. Guia escrito com a equipa real**: rejeitada. É estático e cada troca de
  Pokémon obriga a refazê-lo.

### 6.2 Como a página sabe se uma lure fecha dano

- **1. Regra "quantos Pokémon fecham uma lure"** (ex.: "lura + 2"), indicada por
  hunt.
  - É simples, mas trata todos os Pokémon como iguais.
  - Não consegue equilibrar dano nem mudar o burst de time.
- **2. Pontos de dano + afinação com o uso (escolhida, por ser a mais prática)**.
  - Não é preciso preencher nada antes de caçar.
  - Parte de uma estimativa que dá "lura + 2", como a regra do Mass Roon, e
    afina com os botões faltou/sobrou.
  - Permite equilibrar times como ele faz.
  - Custo: as primeiras lures numa hunt nova podem falhar um pouco.

## 7. O que foi decidido sobre o "como"

### 7.1 Matemática do Disk (núcleo, com testes)

- `R` = cooldown que falta a uma skill quando o Pokémon entra na bola, `N` = intervalo
  do Disk. Tempo `t` na bola até ficar pronta: o menor `t` com
  `t + floor(t / N) >= R`. Com N=3: 60 → 45 s, 40 → 30 s, 30 → 23 s, 20 → 15 s.
- Fora da bola, `R` desce 1 s por segundo.
- Pressuposto (decisão minha): o contador de N s começa quando o Pokémon entra
  na bola. Erro máximo ~1 s; a página mostra "±1 s".
- **Momento de soltar**. As skills de um Pokémon são usadas por ordem, a
  `k × segundos por golpe` depois de o soltar. Ele pode sair quando todas as
  skills marcadas ficam prontas a tempo de serem usadas (na bola acelera, cá
  fora não).
  - A página calcula o momento mais cedo que cumpre isto.
  - Mostra-o como "solta quando <skill mais lenta> marcar N s", que é o número
    que se vê na barra do jogo.

### 7.2 Dados

- **Hunts**: as páginas `Nightmare Hunt - <nome>` dos três clãs e a `Ultra Lab`.
  Campos: nome, dificuldade, resistência, clãs, elemento do dano, elementos
  super efetivos e efetivos, berries, boss/angry/shiny, localização e URL.
- **Pokémon**: das páginas de clã `Orebound`, `Malefic` e `Volcanic`.
  - Por Pokémon: nome, **tier** (T1A, T1B, T1C, T1H, TM, T2, …), elementos e
    função PvE.
  - Da secção "Movimentos PVE" (ou "Moveset PVE") de cada Pokémon, por golpe:
    nome, cooldown (s), etiquetas (Target, AOE, Damage, Stun, Slow, Debuff,
    Buff, …) e elemento.
- **Itens**: Attack Elixir e Nightmare Revive, com os valores da secção 5.
- **Ultra Lab**:
  - as 4 máquinas que cortam 90% do dano;
  - Selfdestruction aos 9%;
  - esquivas de cada Alpha;
  - bónus de held T8 (+40% / −40%);
  - berries Kee, Lum, Red Sitrus e Ganlon;
  - fórmula de equipa e Mecha Device (+10% dano neutro).
- **Referência**: a rotação Pyroar do Mass Roon (1.2) aparece no painel da hunt
  Pyroar.
- Um script gera um JSON a partir da wiki, embutido na página. A página não
  acede à wiki em tempo real. O que o script não conseguir ler é listado, nunca
  inventado.
- Pokémon de outros clãs podem ser adicionados à mão (nome, tier, golpes,
  cooldowns, etiquetas).

### 7.3 Modelo de dano

Fontes de dano por ordem de confiança. A página mostra sempre de qual veio:

1. **Teste próprio** (ecrã "Teste de dano"):
   - Escolhe-se o Pokémon. Testa-se o **combo completo em área**: todas as
     skills AOE com dano, pelas etiquetas da wiki. Escreve-se o dano total que
     **um bicho** levou num alvo neutro. É um número por Pokémon.
   - Segundo número recomendado: o **combo sem a skill de 50 s**, porque as
     rotações ciclam essa skill. A diferença entre os dois é o valor da skill
     de 50 s.
   - O resto divide-se pelas skills proporcionalmente ao cooldown, para usos
     parciais como "só a M8".
   - Skill a skill fica **opcional**, para quem quiser mais precisão; os
     valores por skill sobrepõem-se à divisão.
   - **T1H e Nightmare Move.** O Nightmare Move só funciona na Nightmare World,
     e na Ultra Lab também não funciona. Por isso não se testa em neutro. Para
     um T1H, o ecrã pede:
     - em neutro: o combo sem Nightmare Move, e esse combo também sem a outra
       skill de 50 s;
     - **o Nightmare Move sozinho, numa hunt da Nightmare** onde o elemento
       dele dá dano normal (1×). A página **sugere essas hunts** a partir das
       efetividades.
     - Se for testado numa hunt com outra efetividade, a página divide pelo
       multiplicador para o converter em neutro.
   - As skills de alvo único são opcionais, só para a Ultra Lab.
   - Podem registar-se 2–3 testes por skill; a página usa a média, por causa do
     X-Critical e do device.
   - Registar se o teste foi feito com device, para a página poder separar o
     bónus.
2. **Lista importada**: colar uma tabela de dano de outra fonte (por exemplo, a
   lista de dano do Orebound que o Mass Roon mantém), num formato simples de
   texto/CSV. A tabela de testes também se exporta, para partilhar com o clã.
3. **Por semelhança** (hipótese de quem joga: na PXG o dano é padronizado, e
   Pokémon da mesma função têm o mesmo dano).
   - Cada Pokémon pertence a uma **classe**: tier + função PvE da wiki, e se a
     skill é do elemento do clã ou não. O Mass Roon diz que magias fora do clã
     batem menos.
   - Um teste a um Pokémon dá o dano de cada skill por cooldown (por exemplo,
     T1C burst, skill de 50 s = X). Os outros Pokémon da mesma classe herdam
     esses valores para skills com o mesmo cooldown. Para cooldowns não
     testados, interpola-se pelo cooldown.
   - Marcados "por semelhança".
   - **Verificação da hipótese**: se dois Pokémon testados da mesma classe
     diferirem mais de 10% no mesmo cooldown, a página avisa que a classe não é
     uniforme e deixa de herdar nessa classe.
   - **Lista mínima de testes**: a página sugere os poucos testes que cobrem
     mais classes entre os Pokémon que o boneco tem, priorizando as classes que
     entram nos teams sugeridos.
4. **Estimativa**, quando não há nada melhor: a regra abaixo. Pokémon só com
   estimativa ficam marcados "estimado — testa para confirmar".

**Efetividade na hunt.** O dano é testado em neutro e multiplicado pelo
elemento da skill contra os bichos da hunt.
- Valores da página `Efetividades` da wiki: super efetivo 2×, efetivo 1,75×,
  inefetivo 0,75×, muito inefetivo/nulo 0,5×.
- Usa-se primeiro a lista de super efetivos/efetivos de cada página de hunt.
- Para o resto, a tabela de tipos aplicada aos elementos dos bichos (decisão do
  construtor, com testes).

**Estimativa (fonte 3)**:
- **Dano de uma skill** = cooldown (s) × fator de tipo × fator do Pokémon.
  Decisão minha: quanto maior o cooldown, mais forte o golpe, que é a regra
  geral na PXG.
  - Fator de tipo: AOE com dano = 1; Target com dano = 0,3; sem dano = 0.
  - Fator do Pokémon: por tier, com valores por defeito do construtor e
    editáveis ("este Pokémon bate mais / menos").
  - ×1,15 para o Pokémon com device.
  - O Elixir aplica +35% às skills usadas durante os 20 s seguintes.
- **Alvo da hunt** (dano para fechar uma lure). Por defeito é igual ao combo
  completo de quem lura mais 2 × a mediana dos combos completos dos outros, ou
  seja, a regra "lura + 2".
  - Guardado por boneco e por hunt.
- **Afinação**: "faltou" sobe o alvo 5%, "sobrou" baixa 5%, "fechou" não mexe.
  Pode ser reposto ao valor por defeito.

### 7.4 Escolha da rotação (otimizador)

- **Quem lura**: os Pokémon marcados como "lurador". Por defeito, T1H e
  Pokémon com função de tanque/off-tank; é editável.
- **Uma lure** = um lurador, mais 0 a 3 outros Pokémon, cada um com um conjunto
  de skills. As opções mínimas são "tudo", "tudo menos a skill de 50 s" e "só
  uma skill". Opcionalmente leva Elixir num Pokémon, ou Revive em quem lura
  (que combo de novo).
- **Restrições**:
  - o dano estimado chega ao alvo;
  - cada skill está pronta quando é usada;
  - o elixir e o revive respeitam a sua recarga.
- **Objetivo**: o máximo de lures por hora, ou seja, o mínimo de espera mais
  combate mais deslocação.
  - Desempate: o menor dano a sobrar.
  - Depois: preferências do Mass Roon, como alternar quem lura e guardar a
    skill de 50 s mais forte de quem lura para a lure do elixir.
- **Pesquisa com antecipação**: olha 2–3 lures à frente, para não gastar agora
  o que faz falta já a seguir.
- **Tempos** (editáveis e marcados como estimativa):
  - segundos por golpe (1 s);
  - combate depois do último golpe (2 s);
  - deslocação entre lures, guardada por hunt (10 s).
- **Plano**: simula a partir de tudo pronto e mostra o ciclo que se repete, ou
  as primeiras 12 lures se não houver ciclo limpo.

### 7.4b Escolha do team (team perfeito)

- **Candidatos**: todos os Pokémon da página do clã do boneco (incluindo megas
  e TMs), mais os adicionados à mão.
  - Cada Pokémon tem a marca **"tenho"** por boneco, e o boneco tem um só
    device.
- **Avaliação de um team** = lures/hora que o otimizador de rotação (7.4)
  consegue com ele nessa hunt, com o dano ajustado pela efetividade.
- **Pesquisa**:
  1. Pré-filtrar os candidatos mais fortes por papel (quem lura, burst,
     finalizador) e por efetividade na hunt.
  2. Melhorar por trocas, um Pokémon de cada vez.
  3. Experimentar o device em cada membro.
  - Corre a pedido, com barra de progresso. O tempo e o algoritmo ficam ao
    critério do construtor, mas não pode bloquear a página.
- **Resultados**:
  - melhor team com "tenho";
  - melhor team possível;
  - top 3 compras, com o ganho em lures/hora de cada uma.
  - Cada resultado diz quantos membros têm dano testado ou estimado.

### 7.5 Modo hunt

- Mostra a **lure atual** com os passos por ordem, e em cada passo o
  "solta quando … marcar N s".
- Toca-se **"Recolhi"** quando se recolhe cada Pokémon.
  - Decisão minha: um só toque por Pokémon. O momento em que foi solto
    calcula-se a partir do número de golpes.
  - Elixir e revive têm o seu próprio botão.
- No fim da lure: **Faltou / Fechou / Sobrou**. A página afina e calcula a
  próxima lure a partir dos cooldowns reais.
- Contador de lures e de lures/hora, e botão de reiniciar.
- Funciona bem no telemóvel e pede para o ecrã não apagar.
- Atalhos de teclado opcionais.

### 7.6 Guardar

- Equipas por boneco, marcas "tenho", testes de dano, alvos de dano e tempos
  por hunt ficam no browser (localStorage).
- Há também "Exportar/Importar" por código de texto, para passar entre o PC e o
  telemóvel.

## 8. Deixado ao construtor

- Estrutura do código, nomes, testes e visual. O visual tem de se ler de relance
  durante o jogo, em tema claro e escuro.
- A matemática do Disk, o modelo de dano e o otimizador **têm de ter testes
  automáticos**. Um dos testes reproduz a estrutura da rotação Pyroar do Mass
  Roon com a equipa dele e verifica que o otimizador chega a uma rotação sem
  espera, com 3 Pokémon ou elixir/revive por lure.
- Valores por defeito do fator por tier.
- O algoritmo de pesquisa (feixe, poda, …), desde que responda em menos de ~1 s
  no telemóvel.
- Robustez do parsing do wikitext.

**Ordem de construção sugerida**:
1. Dados da wiki e matemática do Disk.
2. Teste de dano.
3. Otimizador de rotação com o team escolhido à mão, e modo hunt.
4. Team perfeito e compras.
5. Painéis da Ultra Lab.

Assim há algo utilizável cedo.

## 9. Questões em aberto para mais tarde

- Recarga real do Revive na Nightmare (5 min segundo a wiki, ~4 min de ciclo no
  vídeo).
- Se o tick do Disk é por Pokémon ou global (±1 s).
- Rotação automática da Ultra Lab, quando houver vídeo da rotação.
- Obter o link da lista de dano do Mass Roon (comando no chat da live) para
  importar.
- Partilhar a tabela de testes com o clã de forma automática (hoje é
  exportar/importar).
- Mini-jogo dos bosses da Ultra Lab.
