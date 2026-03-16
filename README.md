# ats-parsing-research

> Pesquisa colaborativa sobre como os principais ATS do mercado fazem o parsing de currículos — e o que silenciosamente elimina candidatos bons antes de qualquer humano ver.

Iniciado a partir de uma [discussão no r/brdev](https://www.reddit.com/r/brdev) com contribuições da comunidade.

---

## Por que isso importa

A maioria dos currículos é rejeitada automaticamente por sistemas ATS antes de chegar em um recrutador. O problema não é qualificação — é formato, estrutura e keywords. Este repo documenta o que funciona e o que quebra em cada sistema, com dados empíricos.

---

## ATS cobertos

| ATS | Região | Empresas conhecidas |
|---|---|---|
| Workday | Global | Nubank, iFood, grandes corporações |
| Greenhouse | Global/EUA | Startups americanas, scale-ups |
| Gupy | Brasil | Itaú, Ambev, empresas BR |
| Lever | Global/EUA | Startups de tecnologia |
| Affinda | Global | Parser de terceiros |

---

## Resultados de parsing por formato

### Workday

| Comportamento | Status | Fonte |
|---|---|---|
| Headers e footers ignorados — contato colocado no header some | ❌ Quebra | [TalentTuner (50k+ submissões)](https://talenttuner.app/workday-ats-checker) |
| Datas fora do padrão MM/YYYY quebram parser de experiência silenciosamente | ❌ Quebra | [Rezi](https://www.rezi.ai/posts/why-your-ats-parse-failed) |
| Cada empresa roda instância separada — não existe perfil global | ℹ️ Comportamento | [Teal](https://www.tealhq.com/post/workday-resume) |
| Campos opcionais não preenchidos = campos de busca em branco | ⚠️ Atenção | [Teal](https://www.tealhq.com/post/workday-resume) |

### Greenhouse

| Comportamento | Status | Fonte |
|---|---|---|
| Não usa scoring automático — recrutadores buscam por keywords manualmente | ℹ️ Comportamento | CEO statement |
| Keywords exatas da JD definem quem aparece na busca | ⚠️ Atenção | Análise de fóruns de recrutadores |
| "Monitoramento" ≠ "observability" para o sistema de busca | ❌ Quebra | Análise qualitativa |

### Gupy

| Comportamento | Status | Fonte |
|---|---|---|
| Layout de duas colunas causa dificuldade de parsing | ❌ Quebra | [@Kick_Physical, r/brdev](https://www.reddit.com/r/brdev) — teste empírico |
| Remover colunas resolve o problema de parsing | ✅ Solução | [@Kick_Physical, r/brdev](https://www.reddit.com/r/brdev) — teste empírico |

---

## Resultados por formato de arquivo

Testes realizados com Affinda Resume Parser (trial gratuito em affinda.com).

### Puppeteer (Chromium) vs Google Docs export

Teste conduzido por [@gusbemacbe1989, r/brdev](https://www.reddit.com/r/brdev):

| Campo | Puppeteer | Google Docs | Vencedor |
|---|---|---|---|
| Informações pessoais | ✅ | ✅ | Empate |
| Links no contato (5 links) | ✅ 5/5 | ⚠️ 4/5 | Puppeteer |
| Strings multi-palavra (ex: "Ciência de Dados") | ❌ Quebra no limite do elemento HTML | ✅ Captura completa | Google Docs |
| Nomes de faculdades | ❌ Falha | ✅ Captura | Google Docs |
| Experiência profissional | ✅ | ✅ | Empate |
| Skills e idiomas | ✅ | ✅ | Empate |

**Diagnóstico do problema do Puppeteer:**
- Strings multi-palavra quebram quando o texto está em spans HTML separados — o parser lê o limite do elemento, não a string completa
- Nomes de faculdades com caracteres especiais ou encoding não-padrão falham no Chromium renderer

**Resultado geral: Google Docs vence Puppeteer**

> 🔬 Teste com LaTeX puro em andamento — [@gusbemacbe1989] vai publicar resultados

### Ranking geral de formatos (preliminar)

| Formato | Parsing | Observações |
|---|---|---|
| LaTeX (moderncv, Awesome-CV) | 🟢 Excelente | Texto limpo e selecionável. Evitar multicolunas via minipage/tabular |
| Google Docs → PDF | 🟢 Bom | Skia/PDF renderer. Melhor que ferramentas de design |
| Word → PDF | 🟢 Bom | Estrutura previsível para parsers |
| Puppeteer/Chromium | 🟡 Razoável | Problema com strings multi-palavra e encoding especial |
| Canva / Adobe / Figma | 🔴 Ruim | Texto em camadas ou como imagem — invisível para ATS |
| PDF escaneado | 🔴 Péssimo | Imagem sem texto extraível |

---

## Boas práticas documentadas

### Formato
- [ ] Usar uma coluna — layouts de duas colunas causam problemas em Gupy e outros
- [ ] Datas sempre em MM/YYYY — nunca "Jan/2022" ou só o ano
- [ ] Contato no corpo do documento — nunca em header ou footer
- [ ] Evitar text boxes, tabelas para layout, e elementos de design complexos
- [ ] Preferir Google Docs, Word ou LaTeX para gerar o PDF

### Conteúdo
- [ ] Usar keywords exatas da JD — "observability" não é "monitoramento"
- [ ] Descrever skills com contexto: "3 anos desenvolvendo APIs em Go com foco em observability" > "proficiente em Go"
- [ ] Estrutura de impacto: ação + tecnologia + resultado mensurável
- [ ] Para vagas internacionais: descrever o contexto da empresa brasileira (setor, tamanho, usuários) — recrutador fora do Brasil não conhece a empresa

### Estratégia
- [ ] Apply cirúrgico com currículo ajustado > apply em tudo com currículo genérico
- [ ] Preencher todos os campos opcionais do ATS — cada campo vazio é um critério de busca em branco
- [ ] Manter versão PT-BR e EN-US separadas para mercados diferentes

---

## Como contribuir

Tem dados empíricos sobre algum ATS? Testou um formato e encontrou comportamento não documentado aqui?

1. Abre uma [Issue](../../issues) descrevendo o teste e o resultado
2. Ou abre um PR direto com os dados

**Formato sugerido para contribuição:**

```
ATS: [nome]
Comportamento observado: [descrição]
Como testei: [método]
Resultado: [o que aconteceu]
Fonte/referência: [link ou @usuario]
```

---

## Ferramentas úteis para testar seu currículo

- **[Affinda](https://affinda.com)** — resume parser com trial gratuito, mostra exatamente o que o sistema extrai
- **[Sovren](https://sovren.com)** — parser enterprise com trial
- **[TalentTuner](https://talenttuner.app/workday-ats-checker)** — análise específica para Workday
- **[Teal](https://tealhq.com)** — guias detalhados por ATS

---

## Aplicação prática

Este repo é a base de pesquisa do **[Git Push Hired](https://gitpush-hired.vercel.app)** — ferramenta que usa esses dados para reescrever currículos nos parâmetros do ATS de cada vaga específica. Em desenvolvimento, lista de acesso antecipado aberta.

---

## Licença

MIT — use, contribua, compartilhe.

---

## Contribuidores

Pessoas que contribuíram com dados empíricos, testes reais e perspectivas de dentro do processo:

| Usuário | Contribuição |
|---|---|
| [@gusbemacbe1989](https://www.reddit.com/user/gusbemacbe1989) | Benchmark Puppeteer vs Google Docs no Affinda — dados de parsing por campo |
| [@Kick_Physical](https://www.reddit.com/user/Kick_Physical) | Teste empírico de multicolunas no Gupy — confirmação do problema e solução |
| [@Helltux](https://www.reddit.com/user/Helltux) | Perspectiva de recrutador técnico sênior — como ATS + humano funciona na prática, diferença entre operação e fábrica de software, sugestão de company culture match como feature |

*Discussão original: [r/brdev — Estudei como os principais ATS do mercado funcionam por dentro](https://www.reddit.com/r/brdev)*

---

*Iniciado em março de 2026 · Contribuições bem-vindas*
