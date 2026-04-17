# CS Dashboard - Grupo ROI

Sistema de onboarding e dashboard para a Mentoria ROI (Amazon).

## Estrutura

```
public/
  index.html          # Formulario de onboarding (30 perguntas + LeadScore)
  dashboard.html      # Dashboard com metricas, funil e analytics
  assets/
    roi-avatar.png    # Avatar do Grupo ROI
```

## Como rodar localmente

```bash
cd public
npx http-server -p 8090 -c-1
```

Acesse:
- Formulario: http://localhost:8090/
- Dashboard: http://localhost:8090/dashboard.html

## Funcionalidades

### Formulario de Onboarding
- 30 perguntas com fluxo condicional
- LeadScore (0-100) com classificacao por Tier (A-E)
- Upload de documentos com camera
- CEP autofill via ViaCEP
- Estado/Cidade via API IBGE
- Validacao em tempo real
- Tracking de sessao para funil

### Dashboard
- Cards de resumo por Tier
- Busca rapida (nome, email, WhatsApp, CPF)
- Filtros por data, score e tier
- Oportunidades de upsell CNPJ (CNAEs extras)
- Trending diario/semanal/mensal com grafico de barras
- Funil de conversao screen-by-screen
- Analytics por pergunta
- Quick actions (copiar dados, WhatsApp direto, resumo)
- Export CSV

## Stack atual
- HTML/CSS/JS puro (zero dependencias)
- localStorage para persistencia
- APIs externas: IBGE (estados/cidades), ViaCEP (enderecos)

## Proximos passos
- [ ] Integracao com Supabase (persistencia real)
- [ ] Deploy (Vercel/Netlify)
- [ ] Webhook para HubSpot
- [ ] Notificacoes de lead Tier A/B
