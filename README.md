# Relatórios de criativos: Meta Ads e Instagram orgânico

Dois modelos de relatório HTML que uso no dia a dia de growth e mídia paga, mais a skill que gera esses relatórios com IA.

| Arquivo | O que mostra |
|---|---|
| [`relatorio-criativos-meta.html`](relatorio-criativos-meta.html) | Retro de criativos do Meta Ads: os melhores anúncios por etapa do funil (topo, meio e fundo), cards enxutos com o número principal, por que funcionou e próximo teste, e uma aba com o que evitar no próximo ciclo. |
| [`relatorio-instagram-organico.html`](relatorio-instagram-organico.html) | Top 5 do mês no Instagram orgânico, o que deu certo, o que reduzir, o que testar e uma distribuição editorial sugerida. |
| [`skill/SKILL.md`](skill/SKILL.md) | Instruções para um agente de IA coletar os dados, preencher os modelos e validar o relatório antes da entrega. |

**Ver ao vivo:** [retro de criativos do Meta Ads](https://raulklahold.github.io/relatorios-criativos-portfolio/relatorio-criativos-meta.html) · [Instagram orgânico](https://raulklahold.github.io/relatorios-criativos-portfolio/relatorio-instagram-organico.html)

> A marca "Fôlego", as contas, os criativos e todos os números são **fictícios**. Os frames dos anúncios são ilustrações geradas pelo próprio template.

## Como funciona

Cada HTML é um arquivo único, sem build nem dependências: basta abrir no navegador. Os dados ficam num bloco JSON (`<script id="report-data">`) dentro do arquivo, e o template:

- calcula CTR, CPL/CPA, taxa e custo por view 50%, totais e participação do líder a partir dos números brutos, para nunca haver divergência entre os cards;
- ordena os rankings pelas regras de desempate da metodologia;
- formata tudo no padrão brasileiro (`R$ 1.234,56`, `2,04%`);
- funciona no celular, imprime em A4 e respeita a navegação por teclado.

Para gerar um relatório real, troque o JSON pelos dados da conta e mude `"demo"` para `false`.

---
Raul Klahold, growth e mídia paga
