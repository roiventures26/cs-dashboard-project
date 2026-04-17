# CS Dashboard - Grupo ROI

Dashboard de metricas e analytics para a Mentoria ROI (Amazon).

## Estrutura

```
public/
  dashboard.html      # Dashboard com metricas, funil e analytics
```

## Como rodar localmente

```bash
cd public
npx http-server -p 8091 -c-1
```

Acesse: http://localhost:8091/dashboard.html

## Funcionalidades

- Cards de resumo por Tier (A-E) com percentuais
- Busca rapida (nome, email, WhatsApp, CPF) com highlight
- Filtros por data, score e tier
- Oportunidades de upsell CNPJ (CNAEs extras)
- Trending diario/semanal/mensal com grafico de barras e KPIs
- Funil de conversao screen-by-screen com drop-off
- Analytics por pergunta (distribuicao de respostas)
- Quick actions (copiar dados, WhatsApp direto, resumo, abrir anexos)
- Export CSV

## Stack

- HTML/CSS/JS puro (zero dependencias)
- localStorage para persistencia (migrando para Supabase)

## Repo relacionado

Formulario de Onboarding: https://github.com/roiventures26/cs-onboarding-form
