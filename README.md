# ads-sentinel

Agente de monitoramento de saúde financeira para campanhas do Google Ads — de ponta a ponta.

O ads-sentinel observa continuamente suas campanhas (palavras-chave, dispositivos, locais, públicos, criativos), detecta anomalias e oportunidades, e **envia propostas de ação via Telegram para aprovação humana**. Nada é executado sem o seu "OK".

## O que ele monitora

- **Rentabilidade**: CPA vs. alvo, ROAS, custo por conversão por segmento (palavra-chave, dispositivo, local, campanha)
- **Anomalias**: quedas/picos de CTR, CPC, impressões, conversões vs. baseline histórico
- **Pacing de orçamento**: risco de estouro ou subaproveitamento
- **Integridade**: anúncios reprovados por política, campanhas pausadas inesperadamente, conversões parando de registrar
- **Estrutura**: termos de pesquisa que geram gasto sem conversão (negativação), palavras-chave com Quality Score em queda

## Fluxo de aprovação (Telegram)

```
┌─────────────┐   polling   ┌──────────────┐   análise   ┌─────────────┐
│ Google Ads  │ ──────────► │ Ingestion    │ ──────────► │ Detection   │
│ API (GAQL)  │             │ (raw store)  │             │ Engine      │
└─────────────┘             └──────────────┘             └──────┬──────┘
                                                                │ proposta
                                                                ▼
┌─────────────┐   aprovação  ┌──────────────┐             ┌─────────────┐
│ Você        │ ◄────────── │ Telegram Bot │ ◄────────── │ Dispatcher  │
│ (human-in-  │  (✅ / ❌)   │ (notifica e  │             │ (fila)      │
│  the-loop)  │ ──────────► │  pergunta)   │             └─────────────┘
└─────────────┘   execução   └──────────────┘
```

## Princípios

1. **Human-in-the-loop por padrão** — toda ação sobre a conta passa por aprovação explícita.
2. **Regras antes de ML** — detecção determinística com baseline histórico resolve a maioria dos casos; ML só com dados acumulados.
3. **Sem ruído** — deduplicação de alertas; um incidente = um alerta, com resolução automática.
4. **Custo como requisito** — polling espaçado, respeito às quotas da API.
5. **Decisões documentadas** — ADRs em `docs/adr/`.

## Arquitetura (monorepo)

```
ads-sentinel/
├── docs/
│   └── adr/                      # Architecture Decision Records
├── packages/
│   ├── sentinel-core/            # regras de detecção, baseline, métricas (CPA, ROAS, lucro)
│   ├── sentinel-ingest/          # client Google Ads API, polling, normalização
│   ├── sentinel-notify/          # bot Telegram: notificações + fluxo de aprovação
│   └── sentinel-scheduler/       # jobs agendados, filas, orquestração
├── infra/                        # docker, IaC
└── apps/
    └── api/                      # dashboard (roadmap)
```

## Stack inicial

- **Python 3.12+**, FastAPI, PostgreSQL, Redis
- `google-ads` (client oficial da Google Ads API), GAQL para relatórios
- `python-telegram-bot` para o bot de aprovação
- Docker Compose para dev local

## Roadmap

- [ ] M0 — Ingestão: conectar à API, persistir métricas diárias (campanha/palavra-chave/dispositivo/local)
- [ ] M1 — Bot Telegram: notificações de resumo + fluxo de aprovação
- [ ] M2 — Motor de regras: pacing, anomalias CPC/CTR/conversões, alertas de política
- [ ] M3 — Propostas de ação com aprovação (ex.: "aumentar CPA", "pausar palavra-chave X", "adicionar negativa")
- [ ] M4 — Dashboard web e execução de ações aprovadas
- [ ] M5 — Detecção estatística com sazonalidade (z-score adaptado)

## Começando (dev local)

```bash
git clone git@github.com:<seu-usuario>/ads-sentinel.git
cd ads-sentinel
cp .env.example .env          # preencha credenciais Google Ads + Telegram
docker compose up -d          # PostgreSQL + Redis
# (setup do ambiente Python por pacote — ver packages/*/README.md)
```

## Documentação de decisões

Ver [docs/adr/](docs/adr/).
