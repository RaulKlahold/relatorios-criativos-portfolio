---
name: relatorio-de-criativos
description: Gera relatórios HTML de análise de criativos em dois modos. (A) Mídia paga no Meta Ads, com ranking por etapa do funil, piores criativos e próximos testes. (B) Conteúdo orgânico de um perfil do Instagram, com o Top 5 do mês e a pauta sugerida. Use quando pedirem "análise de criativos", "retro de criativos", "melhores e piores anúncios", "ranking de anúncios por funil", "relatório de criativos do Meta", "top 5 do Instagram", "o que funcionou no orgânico" ou um relatório mensal de conteúdo. Somente leitura: nunca cria, edita, pausa nem publica nada nas contas.
---

# Relatório de criativos: Meta Ads e Instagram orgânico

Esta skill produz dois relatórios HTML autocontidos a partir de dois templates prontos:

| Modo | Template | Responde a |
|---|---|---|
| A, mídia paga | `relatorio-criativos-meta.html` | Quais anúncios puxaram cada etapa do funil, quais perderam eficiência e o que testar a seguir |
| B, orgânico | `relatorio-instagram-organico.html` | Quais posts do mês tiveram mais resposta pública e o que isso sugere para a pauta |

Os templates ficam na raiz deste repositório. **Não reescreva o HTML.** Cada template tem um bloco `<script type="application/json" id="report-data">`, e o trabalho é preencher esse JSON com dados reais. O próprio template calcula CTR, CPL/CPA, taxa de VV50%, custo por VV50%, soma de interações, ordem do ranking e participação do líder. Assim, nenhum número derivado é digitado à mão.

## 0. Antes de começar

1. **Escolha o modo.** Pedido sobre anúncios, campanhas, CPL ou funil → modo A. Pedido sobre posts, perfil, Reels ou orgânico → modo B. Se pedirem "relatório completo" ou os dois, faça A e B. Se não der para saber, pergunte.
2. **Preencha os parâmetros.** Pergunte só o que não estiver no pedido nem nas convenções da conta:

```text
{META_AD_ACCOUNT_IDS}  uma ou mais contas de anúncio (act_...), com um nome curto para cada aba (modo A)
{INSTAGRAM_PROFILE}    perfil sem @ (modo B)
{START_DATE}/{END_DATE} datas inclusivas, AAAA-MM-DD
{TIMEZONE}             padrão America/Sao_Paulo
{TOP_N_PAID}           criativos por etapa; padrão 3
{TOP_N_ORGANIC}        posts no ranking; padrão 5
{MIN_DELIVERY}         entrega mínima para entrar em "o que evitar"; padrão: impressões ≥ 25% da mediana da etapa e investimento ≥ 3% do total da conta
{OUTPUT_DIR}           pasta de saída
```

3. **Convenções da conta.** Se houver um arquivo ou skill de convenções (nomenclatura, metas, evento de conversão por conta, por exemplo `CONFIG-CONTAS.md`), carregue antes de consultar dados. Se não houver, pergunte qual evento conta como resultado de fundo em cada conta.

**Regra de datas.** "Mês atual" significa do dia 1 até ontem (D-1), nunca só o dia de ontem. "Mês passado" ou um mês já encerrado significa o mês inteiro. Sempre mostre as datas exatas no relatório.

## Regras que valem para os dois modos

- Somente leitura. Nada é criado, editado, pausado ou publicado.
- Use apenas dados da plataforma. Não preencha lacunas com estimativa. Se um dado faltar, escreva que faltou.
- Não misture tipos de resultado: lead da plataforma, lead do CRM e lead qualificado são coisas diferentes. Só chame de "qualificado" quando houver cruzamento explícito com a fonte comercial e uma regra de qualificação informada pelo usuário.
- Explicações de desempenho são hipóteses. Use "pode indicar", "o padrão sugere" ou "possível motivo", nunca causalidade afirmada.
- Não invente fala, áudio nem argumento. Sem transcrição, a leitura se limita ao frame, aos textos visíveis, ao nome e à copy.
- Frame, nome e link de cada card apontam para a mesma publicação. Se o post não for localizado, use `"url": null`, e o template mostra "Post não localizado". Nunca fabrique URL.
- Formato brasileiro: `R$ 1.234,56` e `2,04%`, com no máximo duas casas. O template já formata; no JSON, os números vão crus (`1234.56`).
- Nenhum token, cookie, dado pessoal de lead ou credencial entra no HTML nem nos arquivos de apoio.

---

# Modo A: criativos do Meta Ads

## A1. Coleta

Use o conector ou a API do Meta disponível na sessão. Se não houver acesso, peça ao usuário uma exportação do Gerenciador de Anúncios no nível de anúncio, com as colunas abaixo, e siga com ela.

Por conta, no período, **no nível de anúncio**:

- IDs e nomes de campanha, conjunto e anúncio; status;
- investimento, impressões, cliques no link e ações/resultados;
- `video_p50_watched_actions` (views 50%) quando for vídeo;
- identificadores da peça: `video_id`, `image_hash`, `effective_object_story_id`, `instagram_permalink_url`, `thumbnail_url`;
- texto principal, título e descrição, quando disponíveis;
- no conjunto: `optimization_goal` e `promoted_object` (define o evento de conversão).

Faça também uma consulta **no nível da conta** com investimento, impressões, cliques no link e resultados do mesmo período. Esses são os totais dos indicadores do topo (`totals` no JSON).

## A2. Etapa do funil

1. O marcador no nome da **campanha** manda: `[TOPO]`, `[MEIO]`, `[FUNDO]` (ou TOPO, MEIO, FUNDO sem colchetes).
2. Sem marcador, infira só quando for inequívoco: alcance, reconhecimento ou vídeo → topo; tráfego ou engajamento → meio; lead, cadastro ou conversão → fundo.
3. Se continuar ambíguo, a peça fica fora do ranking, e isso vai para a metodologia.

O nome do anúncio pode citar outra etapa (peça reaproveitada). Vale a etapa da campanha. Use a etapa da campanha no campo `stages` e mantenha o nome original do anúncio.

## A3. Consolidação

Antes de ranquear, junte as versões da mesma peça na mesma etapa. A chave de agrupamento segue esta ordem de preferência: `video_id`, depois `image_hash`, depois o post efetivo e, só em último caso, o nome do anúncio normalizado. Some investimento, impressões, cliques, views 50% e resultados das versões.

## A4. Ranking

| Etapa | Ordena por | Desempate |
|---|---|---|
| Topo e meio | views 50% (quantidade), decrescente | menor custo por view 50%, depois nome |
| Fundo | resultados do evento configurado, decrescente | menor custo por resultado, depois nome |

- No fundo, use **só** o evento do `promoted_object` do conjunto. Se a mesma conversão aparecer como evento padrão, pixel e conversão personalizada, conte uma vez e explique a escolha na metodologia.
- Imagem estática no topo ou no meio não tem views 50%. Ranqueie essas peças separadamente por CTR de link ou deixe-as fora, e registre a decisão.

## A5. O que evitar

Em cada conta, escolha até 3 peças com eficiência abaixo dos pares **da mesma etapa** e entrega acima de `{MIN_DELIVERY}`:

- topo e meio: menor taxa de VV50%;
- fundo: maior custo por resultado.

Peça com pouca entrega não é "pior", é inconclusiva: deixe de fora. Peça feita para um recorte específico (público pequeno, região, nicho) não se compara com peças amplas. Ela entra com `"kind": "context"`, recebe o selo `Contexto específico` e as notas `contexto`, `leitura` e `uso`, em vez de `sinal`, `motivo` e `evitar`.

Feche a aba com 3 regras curtas (`rules` no JSON) que resumam o aprendizado do mês.

## A6. Texto de cada card

Todo vencedor tem as três notas, com frases curtas e concretas:

- **Leitura** (`leitura`): o que a peça comunica, para quem e com qual promessa.
- **Hook** (`hook`): o que abre ou sustenta a atenção (fala, texto na tela, cena, pergunta, número, personagem).
- **Próximo teste** (`teste`): uma variação produzível e mensurável. "Testar outro criativo" não serve; "abrir com a tela do app e manter o mesmo personagem e CTA" serve.

## A7. Preenchimento do JSON (modo A)

Copie `relatorio-criativos-meta.html` para `{OUTPUT_DIR}/relatorio-criativos-meta.html` e substitua o bloco `report-data`. Campos principais:

```json
{
  "demo": false,
  "author": "nome de quem assina",
  "client": "nome do cliente", "clientDescription": "o que ele faz, em poucas palavras",
  "title": "Retro de criativos", "subtitle": "...", "periodLabel": "novembro de 2026",
  "start": "AAAA-MM-DD", "end": "AAAA-MM-DD", "extractedAt": "AAAA-MM-DD", "timezone": "America/Sao_Paulo", "currency": "BRL",
  "accounts": [{
    "id": "curto-sem-espaco", "tab": "rótulo da aba", "title": "...", "audience": "...", "accountId": "act_...",
    "resultLabel": "leads", "resultSingular": "lead", "costLabel": "CPL", "conversionEvent": "nome do evento",
    "totals": {"spend": 0, "impressions": 0, "clicks": 0, "results": 0},
    "stages": {"topo": [], "meio": [], "fundo": []},
    "worst": []
  }],
  "rules": [{"big": "...", "small": "..."}]
}
```

Cada criativo leva `name`, `format` (`VID`, `IMG` ou `CAR`), `url`, `thumb`, `badges`, `spend`, `impressions`, `clicks`, `vv50` (topo/meio) ou `results` (fundo) e `notes`. Em `worst`, inclua também `stage` e `kind` (`alert` ou `context`).

- `thumb`: prefira incorporar o frame como data URI (JPEG, cerca de 600 px de largura), porque links de thumbnail do Meta expiram. Com `thumb: null`, o template desenha um frame ilustrativo usando `frameText`; isso serve para modelo, não para relatório real.
- `badges`: só quando ajudarem a ler (`Maior volume`, `Melhor taxa`, `Melhor CPL`, `Pouca entrega`, `Contexto específico`).
- Uma conta vira uma aba. Com uma conta só, ainda há duas abas (a conta e "O que evitar").

---

# Modo B: Instagram orgânico

## B1. Coleta

- Liste **todos** os posts publicados entre `{START_DATE}` e `{END_DATE}`, não só os do último dia.
- Posts fixados de meses anteriores e anúncios ficam fora do ranking.
- Para cada post, colete a URL, a data, o formato (Reels, Carrossel ou Foto), uma legenda curta, o frame, as curtidas e os comentários.
- Com acesso autenticado aos insights, colete também alcance, salvamentos, compartilhamentos e visitas ao perfil, e marque `"source": "insights"`. Sem esse acesso, use `"source": "public"`.

## B2. Ranking

- Métrica: **interações públicas = curtidas + comentários**. Não chame isso de "engajamento", porque salvamentos e compartilhamentos não estão incluídos.
- Desempate: mais comentários, depois mais curtidas, depois o post mais recente.
- O template já filtra pelo período, ordena, corta o Top N e calcula os totais. Basta colocar todos os posts do período em `posts`.
- Com menos de 5 posts no período, o template mostra os que existem e explica a falta.
- Posts publicados nas últimas 48 h antes de `{END_DATE}` tiveram menos tempo de exposição. Se um deles ficar fora do Top 5 por pouco, cite isso numa das notas de aprendizado.

## B3. Leitura editorial

- `learnings`: de 2 a 4 padrões observáveis nos posts do topo, cada um com um exemplo do ranking (pessoa com nome e história, bastidor real, dúvida concreta respondida, número ou prazo, prova social).
- `reduce`: padrões a diminuir, não proibições (benefícios genéricos empilhados, marca antes da história, CTA repetido, muitos conceitos num post só).
- `test`: hipóteses executáveis para o mês seguinte.
- `mix`: distribuição editorial sugerida somando 100%. Inclua só quando houver base (pelo menos 8 posts). Sem base, deixe `"mix": []`, e a seção some.

## B4. Preenchimento do JSON (modo B)

Copie `relatorio-instagram-organico.html` para `{OUTPUT_DIR}/relatorio-instagram-organico.html` e substitua o bloco `report-data`. Os campos são `demo`, `author`, `brand`, `profile`, `followers`, `title`, `subtitle`, `monthLabel`, `start`, `end`, `collectedAt`, `topN`, `source`, `posts[]`, `learnings[]`, `reduce[]`, `test[]` e `mix[]`. Use `thumb` com o frame real do post; nunca uma imagem genérica.

---

# Entrega e validação

## Arquivos em `{OUTPUT_DIR}`

```text
relatorio-criativos-meta.html       modo A
relatorio-instagram-organico.html   modo B
dados-fonte.json                    o mesmo JSON que foi para o(s) HTML(s), mais os dados brutos por anúncio/post
validacao.json                      resultado de cada checagem abaixo (ok/falhou + valores)
```

## Checagens obrigatórias

Dados:
- [ ] O período cobre exatamente `{START_DATE}` a `{END_DATE}` (e "mês atual" foi até D-1).
- [ ] A soma do investimento e das impressões de todos os anúncios bate com o total da conta (tolerância de R$ 0,01 e de 0 impressões).
- [ ] O evento de fundo de cada conta é o do `promoted_object`, contado uma vez só.
- [ ] Nenhuma peça aparece duas vezes na mesma etapa (consolidação feita).
- [ ] No modo B, todos os posts do período estão em `posts`.

Links e imagens:
- [ ] Cada `url` é do post certo (confira o ID do post contra o anúncio ou o perfil). Links do Instagram e do Facebook podem exigir login para abrir. Isso não é erro; o que não pode é URL inventada.
- [ ] Nenhuma imagem quebrada e nenhum caractere `�`.

Visual (abrir no navegador):
- [ ] Captura em desktop (cerca de 1366 px) e em celular (375 px).
- [ ] Sem rolagem horizontal: `document.documentElement.scrollWidth <= innerWidth`.
- [ ] Console sem erros de JavaScript.
- [ ] `"demo": false` (senão aparece o selo "Modelo com dados fictícios").

## Concluído quando

1. O recorte de datas está certo e os totais foram reconciliados.
2. Os rankings usam métricas comparáveis dentro de cada etapa.
3. Cada card tem frame, nome e link do mesmo anúncio ou post, mais as três notas.
4. O relatório orgânico mostra o Top 5 do mês em página única.
5. Desktop e celular foram conferidos por captura de tela.
6. `validacao.json` não tem nenhuma checagem com falha, ou as falhas foram explicadas ao usuário.
