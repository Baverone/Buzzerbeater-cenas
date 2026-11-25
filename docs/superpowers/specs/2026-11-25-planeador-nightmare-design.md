# Planeador de Rotações Nightmare (PokeXGames) — Desenho

Data: 2026-11-25 · Estado: rascunho para aprovação

## 1. Intenção e porquê

Quem vai usar isto joga PokeXGames (PXG) com vários bonecos, todos com
**Nightmare Disk 4.0**. O objetivo é caçar nas hunts da Nightmare World "100%
eficiente": nunca ficar parado à espera de cooldowns entre lures, sabendo em
cada momento que Pokémon soltar, que golpes usar e quantos segundos falta para o
kit estar pronto.

Mecânica central (confirmada por quem joga e pela wiki):

- O Disk só atua **com o Pokémon dentro da Pokébola**. Fora da bola o cooldown
  desce ao ritmo normal (1 s por segundo).
- Dentro da bola, além do ritmo normal, recupera **1 s extra a cada N s**:
  Disk 1.0 → N=8, 2.0 → N=6, 3.0 → N=4, **4.0 → N=3**.
- Os helds usados são **X-Attack e X-Critical**, nunca X-Cooldown. Os cooldowns
  base são os da wiki, sem redução.

A forma de tirar partido disto é uma rotação de trocas: enquanto um Pokémon
ataca uma lure, os outros recuperam na bola com o Disk.

Prioridade de clãs: **Orebound → Malefic → Volcanic**. O boneco principal é
Orebound (Rebound); os outros dois clãs são de outros bonecos.

## 2. Objetivos

1. Uma página (planeador) onde se escolhe **clã → hunt → equipa** e se obtém a
   rotação: ordem de soltar, golpes por ordem em cada lure, espera (s) antes de
   soltar e momento em que cada Pokémon fica pronto. Também diz se a equipa chega
   para zero espera e, se não chegar, quanto se espera por ciclo.
2. Um **modo hunt** (cronómetro ao vivo) para usar durante o jogo. Toca-se ao
   soltar e ao recolher cada Pokémon e a página conta, com a matemática do Disk,
   quanto falta para cada um estar pronto, indicando o próximo a soltar.
3. Para cada hunt, um **painel de informação** tirado da wiki: elemento do dano
   dos bichos, elementos super efetivos e efetivos contra eles, berries
   recomendadas, boss/angry/shiny do local, resistência e caminho, e link para a
   página da wiki.
4. Cobrir as hunts indicadas para Orebound (27), Malefic (6) e Volcanic (20)
   segundo a wiki, mais a **Ultra Lab** como entrada especial.
5. Equipas guardadas **por boneco**, para não ter de as escolher de cada vez.

## 3. Não-objetivos (fica para depois ou fora)

- **Mini-jogo de treino dos bosses da Ultra Lab**: projeto separado, a seguir a
  este.
- **Análise de vídeos** (Masrune, Artilheiro) para tirar rotações e posições de
  lures: YouTube e Twitch estão bloqueados nesta sessão. Fica para quando houver
  links concretos. O planeador tem de funcionar sem isso.
- Posições das lures no mapa e mapas desenhados. A wiki não as tem; os
  minimapas dizem que os Pokémon "não representam o local exato".
- Hunts de clãs que não sejam estes três.

## 4. Anti-objetivos (seria um fracasso mesmo que "funcionasse")

- **Qualquer coisa que jogue por quem joga**: enviar teclas ou cliques ao jogo,
  ler a memória ou o ecrã do cliente, macros. Isso dá ban. O planeador é só
  informação e cronómetro, operado à mão.
- Apresentar como exatos números que são estimativas, como o tempo de cada golpe
  ou o tempo de combate numa lure. Têm de aparecer como valores ajustáveis.
- Assumir uma equipa que não existe. A equipa é sempre escolhida na página; não
  há equipa "padrão" imposta.

## 5. Restrições

- Dados vêm da wiki oficial (`wiki.pokexgames.com`), em português. A rede da
  sessão já permite este host. A wiki pode estar desatualizada; cada dado mostra
  a página de origem.
- O jogo tem o foco do teclado durante a hunt. Atalhos de teclado na página só
  funcionam se ela tiver o foco (2.º monitor clicado). Por isso o modo hunt tem
  de funcionar bem **com toques no telemóvel/tablet ao lado do PC**, com botões
  grandes.
- Entrega como página web única (HTML + JS + dados embutidos), publicada com
  link privado para abrir no PC ou no telemóvel. O código fonte e o script de
  recolha de dados ficam no repositório, no ramo `claude/sleepy-feynman-75lqyr`.
- Interface em português de Portugal.

## 6. Abordagens consideradas

- **A. Planeador interativo (a escolhida).** A página calcula a rotação a partir
  da equipa escolhida e dos cooldowns da wiki, e tem modo hunt. Funciona para
  qualquer equipa e para os três bonecos, e recalcula quando se troca de
  Pokémon. O custo é que o tempo entre lures é introduzido à mão.
- **B. Guia escrito com equipa "padrão" por clã.** Lê-se e segue-se, mas a equipa
  é suposta. Seriam mais de 50 hunts de texto quase repetido, e o guia estaria
  errado se a equipa real fosse outra.
- **C. Guia escrito com a equipa real.** Exato, mas estático: cada troca de
  Pokémon obriga a refazê-lo.

Escolhida a A porque é a única que serve os três bonecos sem adivinhar equipas.

## 7. O que foi decidido sobre o "como"

### 7.1 Matemática do Disk (núcleo, tem de ter testes)

- Seja `R` o cooldown que falta a um golpe no momento em que o Pokémon é
  recolhido, e `N` o intervalo do Disk.
- Tempo `t` dentro da bola até ficar pronto: o menor `t` inteiro com
  `t + floor(t / N) >= R`.
  Exemplos com N=3: 60 s → 45 s; 40 s → 30 s; 30 s → 23 s; 20 s → 15 s.
- Fora da bola, `R` desce 1 s por segundo.
- **Pressuposto (decisão minha):** o contador de N s começa no momento em que o
  Pokémon entra na bola. Não se sabe se o jogo usa um relógio global, e o erro
  máximo é de ~1 tick (1 s). A página mostra uma nota "±1 s".
- Golpes que ficam prontos antes de o Pokémon ser recolhido não contam para a
  espera.

### 7.2 Dados

- **Hunts** (das páginas `Nightmare Hunt - <nome>` cujos "Clãs indicados"
  incluem Orebound, Malefic ou Volcanic, mais `Ultra Lab`): nome, dificuldade
  (Baixa/Alta), resistência, clãs indicados, elemento do dano dos bichos,
  elementos super efetivos e efetivos, berries/elixires recomendados, notas
  (boss, angry, shiny), texto de localização e URL da wiki.
- **Pokémon** (das páginas de clã `Orebound`, `Malefic`, `Volcanic`, por tier):
  nome, tier, elementos, função PvE (dano explosivo, dano contínuo, etc.) e,
  da página de cada Pokémon, a secção **"Movimentos PVE"**. Para cada golpe:
  nome, cooldown (s), etiquetas (Target, AOE, Damage, Stun, Slow, Debuff, Buff,
  ...) e elemento.
- **Ultra Lab**: notas dos Alpha Subject #1–#4.
  - Partir primeiro as 4 máquinas de defesa, que cortam 90% do dano.
  - Selfdestruction aos 9% de HP.
  - Esquivas:
    - Psyusion: subir para o SQM do boss.
    - Stickmerang/Brave Bird: recolher o Pokémon antes do fim da canalização.
    - Wild Charge: ficar à frente.
    - Bone Rush: ficar atrás.
  - Bónus de Held X Tier 8: +40% de dano e −40% de dano recebido.
  - Berries: Kee, Lum, Red Sitrus e Ganlon.
- Um script de recolha gera um ficheiro de dados (JSON) a partir da wiki, e esse
  ficheiro é embutido na página. A recolha corre quando for preciso atualizar; a
  página não acede à wiki em tempo real.
- **Pokémon fora destes clãs:** a página permite adicionar um Pokémon à mão
  (nome + golpes + cooldowns + etiquetas), porque se usam Pokémon de outros
  clãs.

### 7.3 Planeador (plano estático)

Entradas:
- Boneco (nome livre) e clã. Hunt (filtrada pelo clã, pesquisável).
- Disk: 1.0–4.0, **4.0 por defeito**.
- Equipa: 1 a 6 Pokémon, por ordem de rotação.
- Golpes de cada Pokémon a usar numa lure: por defeito os de **AOE**; cada um
  pode ser ligado ou desligado. A ordem dentro da lure segue a regra por defeito
  descrita abaixo e pode ser reordenada.
- Tempos, todos editáveis e marcados como estimativas:
  - "segundos por golpe" (por defeito 1 s);
  - "segundos de combate na lure depois do último golpe até recolher" (por
    defeito 2 s);
  - "segundos entre lures": de recolher até chegar à lure seguinte e soltar.
    Guardado por hunt, por defeito 10 s.

Saída:
- Tabela por lure (simular pelo menos 2 voltas completas da equipa): n.º da
  lure, Pokémon a soltar, golpes por ordem, **espera antes de soltar (s)**, e
  quando esse Pokémon volta a estar pronto.
- Resumo: "Zero espera" ou "Esperas X s por volta". Inclui a **equipa mínima**
  (quantos Pokémon da ordem atual chegam para zero espera) e o **tempo entre
  lures mínimo** para zero espera com a equipa atual.
- Por defeito a rotação é round-robin pela ordem da equipa.
  **Decisão minha:** se o próximo da ordem não estiver pronto e outro estiver,
  sugerir o que está pronto (marcado como "troca sugerida").
- **Ordem dos golpes por defeito (decisão minha):** primeiro os de controlo
  (Stun, Slow, Debuff, Buff), depois os de dano AOE do cooldown maior para o
  menor. Pode ser reordenada à vontade.

### 7.4 Modo hunt (cronómetro ao vivo)

- Um cartão grande por Pokémon da equipa, com estado **na bola / fora** e
  contagem decrescente até ao kit estar pronto (verde quando pronto).
- Toque em "Soltar": arranca os cooldowns dos golpes selecionados, espaçados por
  "segundos por golpe". Toque em "Recolher": a partir daí a contagem passa a usar
  a regra do Disk.
- Destaque do **próximo a soltar**, contador de lures e botão de reiniciar.
- Atalhos de teclado 1–6 (soltar/recolher) para quem usa num 2.º monitor com
  foco.
- Funciona bem a largura de telemóvel e não deixa o ecrã apagar enquanto está
  ativo, se o browser permitir.

### 7.5 Guardar

- Equipas por boneco, tempos por hunt e preferências guardados no próprio
  browser (localStorage). Há também "Exportar/Importar" num código de texto para
  passar do PC para o telemóvel.
- **Decisão minha:** localStorage chega, porque é só um utilizador. Sincronizar
  automaticamente entre dispositivos fica para depois, se for preciso.

## 8. Deixado ao construtor

- Estrutura do código, nomes de ficheiros e testes. A matemática do Disk e o
  simulador do plano têm de ter testes automáticos.
- Linguagem do script de recolha e robustez do parsing do wikitext. Há páginas
  com formatação diferente; o que falhar deve ser listado, não inventado.
- Visual da página, desde que seja legível de relance durante o jogo
  (contraste alto, números grandes) e funcione em tema claro e escuro.
- Como mostrar a Ultra Lab na lista (entrada especial no topo, por exemplo).

## 9. Questões em aberto para mais tarde

- Posições das lures e tempos reais entre lures por hunt, a partir de vídeos
  enviados com links concretos.
- Confirmar no jogo se o tick do Disk é por Pokémon ou global (afeta ±1 s).
- Mini-jogo dos bosses da Ultra Lab (próximo projeto).
