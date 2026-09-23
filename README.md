# GridIron-IQ
Programa para sugestão de jogadas para os treinadores
# 🏈 GridIron IQ

**Inteligência de jogo para treinadores — decisão de jogada e efetividade de jogadores.**

GridIron IQ é uma ferramenta de apoio à decisão para o treinador/analista
escolher a jogada mais efetiva em uma dada situação de jogo e avaliar a
efetividade dos jogadores, com base no histórico do NFL Big Data Bowl
(temporada 2021, Semanas 1-8).

O nome combina *gridiron* (apelido clássico do campo de futebol americano)
com *IQ* (a "inteligência de jogo" que orienta as decisões do treinador).

## Visão geral

GridIron IQ transforma os dados do **NFL Big Data Bowl 2021** (Semanas 1-8)
em recomendações de jogada e avaliações de jogadores, respondendo à pergunta
prática: *"nesta situação, o que historicamente funcionou melhor — e quais
jogadores importam?"*

### Base de dados

Usa três arquivos principais do dataset, ligados por IDs:

| Arquivo | Papel |
|---|---|
| `plays.csv` | Situação e resultado de cada jogada (base das recomendações) |
| `pffScoutingData.csv` | Desempenho individual por jogada (pressão / proteção) |
| `players.csv` | Nome, posição e físico dos jogadores |

Complementado por `games.csv` (semana/time) e por headshots públicos da
nflverse (fotos dos jogadores).

### O que a aplicação faz

1. **Recomendação de jogada por situação** — o usuário define down, jardas,
   região do campo e cobertura da defesa; o sistema busca jogadas históricas
   semelhantes e ranqueia combinações de formação × dropback × play-action
   por um score de efetividade (jardas + bônus por converter − penalização
   por sack/interceptação). Com amostra pequena, afrouxa os filtros
   automaticamente.
2. **Filtro de contexto** — barra horizontal no topo para filtrar por semana
   e por time antes de qualquer análise.
3. **Busca de jogada** — seleção de uma combinação específica para ver
   métricas e exemplos reais de jogadas.
4. **Painel de jogadores** — rankings de pass rush e de proteção, com foto e
   perfil no hover.
5. **Busca de jogador com avaliação em estrelas** — foto, perfil e um cartão
   de 5 características por função (0,5-5 estrelas, por percentil na posição)
   com gráfico radar e explicação de cada atributo no hover.
6. **Campo interativo por posição** — clique numa posição no campo para ver
   os principais jogadores dela, com foto e métrica.

### Interface

- Seletor de idioma **PT-BR / EN-US** (apenas seleção, sem digitação).
- Tema escuro (fundo `#091026`) inspirado na identidade da NFL.
- Filtros no topo, gráficos interativos e tooltips explicativos.

### Limitações

- As métricas são **descritivas** (o que aconteceu no histórico), não
  previsões causais — apoio à decisão, não substituto do julgamento do
  treinador.
- O dataset cobre bem **pass rush e proteção**; por isso ratings e rankings
  existem para essas funções, enquanto posições como QB/WR/CB aparecem sem
  avaliação por falta de métricas.
- Recorte temporal: temporada 2021, Semanas 1-8.

## Como funciona

O treinador informa a situação (down, jardas para o first down e,
opcionalmente, região do campo e a cobertura da defesa). A ferramenta:

1. Filtra no histórico as jogadas semelhantes àquela situação.
2. Ranqueia as combinações de **formação × tipo de dropback × play-action**
   por um score de efetividade da jogada.
3. Complementa com o **painel de jogadores**: quem mais pressiona o QB
   (reforçar proteção) e quais bloqueadores são mais confiáveis.

### Score de efetividade da jogada (`play_score`)

Combina o resultado real com o risco, em escala aproximada de "jardas":

```
score = jardas ganhas
        + 2   se a jogada foi bem-sucedida (regra de sucesso por down)
        - 6   se resultou em sack
        - 12  se resultou em interceptação
```

Regra de sucesso da jogada (padrão da análise de futebol americano):
- 1º down: ganhar >= 40% das jardas necessárias
- 2º down: ganhar >= 60%
- 3º/4º down: converter o first down (>= 100%)

Só combinações com amostra mínima (>= 8 jogadas) são recomendadas. Se a
situação exata tiver poucos dados, o filtro afrouxa progressivamente
(ignora região do campo, depois cobertura, depois usa só o down).

## Requisitos

- Python 3 (testado no 3.14)
- pandas (`py -m pip install pandas`)
- Para o dashboard: streamlit e plotly (`py -m pip install streamlit plotly`)

## Dashboard interativo (com gráficos e filtro por semana/time)

Além da CLI, há um painel visual em `dashboard.py` (Streamlit + Plotly):

```powershell
py -m streamlit run dashboard.py
```

Abre no navegador (`http://localhost:8501`). Recursos:

- **Filtro por semana** (uma ou várias) e **por time no ataque** — junta `games.csv`.
- Seleção da situação (down, jardas, região do campo, cobertura da defesa).
- Tabela de jogadas recomendadas + gráfico de efetividade por jogada.
- Painel de jogadores com gráficos: ameaças de pass rush e proteção mais confiável.
- **Barra de pesquisa de jogada**: selecione formação, tipo de dropback e
  play-action para ver o desempenho daquela combinação (nº de jogadas, score,
  jardas, sucesso, sack) e exemplos reais com a descrição da jogada.
- **Barra de pesquisa de jogador**: digite ou selecione um nome e veja o
  perfil (nascimento/idade, altura, peso, faculdade) e uma **avaliação em
  estrelas** (5 características por função, com gráfico radar). Os números
  crus ficam num expander abaixo.

## Avaliação em estrelas (rating por função)

Para dar uma leitura rápida de treinador/analista, cada jogador recebe um
rating de **0,5 a 5 estrelas** em 5 características, calculado por **percentil
dentro da própria função** (5★ = elite do grupo, não um valor absoluto):

- **Pass rusher**: Pressão · Finalização (sack) · Impacto (hit) ·
  Perturbação (hurry) · Volume/Durabilidade.
- **Bloqueador**: Confiabilidade · Proteção de sack · Proteção de hit ·
  Proteção de hurry · Volume/Durabilidade (para "proteção", menos permitido
  = mais estrelas).

Só jogadores com um mínimo de jogadas recebem estrelas; abaixo disso, apenas
os números crus são exibidos. Como o dataset cobre pass rush e proteção, o
rating existe para essas duas funções.

## Fotos dos jogadores (headshots / media day)

As fotos são os headshots oficiais da NFL, obtidos via nflverse (dados
públicos) e casados por nome com o `players.csv`. O mapeamento fica no
arquivo local `headshots.csv` (nflId, displayName, headshot_url).

Para (re)gerar o mapeamento (precisa de internet):

```powershell
py build_headshots.py
```

Cobertura atual: 1.660 de 1.679 jogadores (99%) com foto. Quem não tem
headshot recebe um avatar com as iniciais (placeholder neutro).

Onde as fotos aparecem no dashboard:
- No **hover** das barras dos rankings de pass rush e proteção.
- Ao lado do perfil quando o jogador é **selecionado na busca**.

Observação de direitos: as imagens pertencem à NFL/nflverse; aqui são usadas
apenas para exibição no painel de análise.

## Campo interativo por posição

O dashboard tem um campo de futebol americano desenhado (gramado, linhas de
jarda e linha de scrimmage) com as posições dispostas como numa formação:
ataque na metade inferior (azul) e defesa na superior (vermelho).

Ao **clicar em uma posição**, aparecem os principais jogadores dela, com
foto, nome e a métrica mais relevante:
- Posições de pass rush (DE, DT, NT, OLB, ILB, MLB) → ordenadas por taxa de
  pressão ao QB.
- Posições de bloqueio (T, G, C, TE, RB, FB) → ordenadas por confiabilidade
  na proteção.

Posições como QB, WR e CB aparecem no campo, mas o dataset (focado em pass
rush e proteção) não tem métricas de efetividade para elas — nesse caso é
exibida uma mensagem informativa.

## Idioma (PT-BR / EN-US)

No canto superior direito há um seletor 🌐 para alternar a interface entre
**PT-BR** (padrão) e **EN-US**. Toda a interface (títulos, filtros, rótulos,
mensagens, atributos das estrelas e textos do campo) é traduzida. As
traduções ficam centralizadas em `i18n.py` — para ajustar um texto ou
acrescentar um idioma, basta editar esse arquivo.

Observação: nomes próprios do dataset (jogadores, times, formações como
SHOTGUN, coberturas como Cover-3) permanecem no original, pois são termos
técnicos/próprios do futebol americano.

## Tema visual

Paleta inspirada na NFL (azul-marinho de fundo, detalhes em vermelho suave e
prata, texto branco), definida em `.streamlit/config.toml` e refinada com CSS
no `dashboard.py`. Os gráficos de estatística mantêm as escalas verde
(jogadas), vermelho (pass rush) e azul (proteção).

O motor de recomendação é o mesmo da CLI, aplicado sobre o subconjunto
filtrado. Se a combinação semana+time+situação tiver poucos dados, o filtro
afrouxa automaticamente (o painel mostra qual base foi usada).

## Uso

Rode a partir da pasta `coach_tool` (no Windows use `py`):

```powershell
# 3º down e longa (8+ jardas) contra Cover-3
py coach_cli.py --down 3 --ytg 8 --coverage Cover-3

# 1º e 10 na red zone (yardline = distância até a end zone)
py coach_cli.py --down 1 --ytg 10 --yardline 15

# Melhor ataque geral contra uma cobertura
py coach_cli.py --somente-cobertura --coverage Cover-2

# Apenas o painel de jogadores
py coach_cli.py --jogadores
```

### Parâmetros

| Flag | Descrição |
|---|---|
| `--down` | Descida (1-4) |
| `--ytg` | Jardas para o first down |
| `--yardline` | `absoluteYardlineNumber` (distância até a end zone; menor = red zone). Opcional |
| `--coverage` | Cobertura da defesa: `Cover-0/1/2/3/6`, `2-Man`, `Quarters`, etc. Opcional |
| `--top` | Quantas jogadas listar (padrão 5) |
| `--somente-cobertura` | Ignora a situação e mostra o que funciona vs a cobertura (exige `--coverage`) |
| `--jogadores` | Mostra apenas o painel de efetividade de jogadores |

### Exemplo de saída (3º e longa vs Cover-3)

```
#  Formação    Dropback        PlayAct N    Score   Jardas  Suc%   Sack%  INT%
1  EMPTY       TRADITIONAL     nao     30   9.27    9.80    33%    7%     7%
2  EMPTY       SCRAMBLE        nao     14   7.43    8.43    36%    29%    0%
3  SHOTGUN     SCRAMBLE        nao     35   6.06    6.17    29%    6%     3%
4  SHOTGUN     TRADITIONAL     nao    175   4.90    5.70    23%    11%    5%
```

## Estrutura do código

| Arquivo | Responsabilidade |
|---|---|
| `data_prep.py` | Carrega os CSVs, cria buckets de situação, métricas de efetividade (jogada e jogadores) e o contexto de jogo (semana/time via `games.csv`) |
| `recommender.py` | `PlayRecommender`: filtra por situação e ranqueia as jogadas |
| `players_layer.py` | `PlayersLayer`: efetividade individual (pass rush e proteção) |
| `coach_cli.py` | Interface de linha de comando que amarra tudo |
| `dashboard.py` | Painel visual interativo (Streamlit + Plotly) com filtro por semana/time e gráficos |

## Perfil dos jogadores nas estatísticas

Cada jogador que aparece nas estatísticas (pass rush e proteção) tem um
perfil vindo de `players.csv`:

- **Data de nascimento** e **idade** (calculada na data de referência
  2021-09-01, início da temporada).
- **Altura** (formato pés-polegadas, ex.: `6-4`), **peso** (libras) e **faculdade**.
- Detalhes da posição que não estão no gráfico principal: para pass rushers,
  sacks/hits/hurries; para bloqueadores, sacks/hits/hurries permitidos.

No **dashboard**, esses dados aparecem **apenas ao passar o mouse** sobre a
barra do jogador (hover), mantendo o gráfico principal limpo.

Na **CLI** (que é texto e não tem hover), o perfil fica oculto por padrão e
só aparece com a flag `--perfil`:

```powershell
py coach_cli.py --jogadores --perfil
```

## Limitações

- O dataset é focado em **proteção de passe / pass rush**, então as
  recomendações cobrem bem jogadas de passe/dropback. Jogadas de corrida
  não têm o mesmo detalhamento aqui.
- As métricas são descritivas (o que funcionou no histórico), não uma
  previsão causal. Servem como apoio, não substituem o julgamento do treinador.
- Amostras pequenas em situações muito específicas são sinalizadas pelo
  afrouxamento automático dos filtros e pela coluna `N` (nº de jogadas).
```
