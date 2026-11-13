# AI Sales Automation Pipeline

> **Pipeline multi-agente autônomo que prospecta, diagnostica e fecha o ciclo de vendas — do Google Maps ao WhatsApp, sem intervenção humana antes da aprovação final.**

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=node.js&logoColor=white"/>
  <img src="https://img.shields.io/badge/TypeScript-5.4-3178C6?style=for-the-badge&logo=typescript&logoColor=white"/>
  <img src="https://img.shields.io/badge/Claude_AI-Anthropic-D97706?style=for-the-badge&logo=anthropic&logoColor=white"/>
  <img src="https://img.shields.io/badge/WhatsApp-API-25D366?style=for-the-badge&logo=whatsapp&logoColor=white"/>
  <img src="https://img.shields.io/badge/Gmail-OAuth_2.0-EA4335?style=for-the-badge&logo=gmail&logoColor=white"/>
  <img src="https://img.shields.io/badge/PM2-Process_Manager-2B037A?style=for-the-badge&logo=pm2&logoColor=white"/>
</p>

---

## O que é

Sistema de automação de prospecção B2B construído como referência de arquitetura multi-agente. Roda automaticamente às 08h, executa todo o funil de vendas e entrega resultados no painel de aprovação — o vendedor só decide o que enviar.

Cada agente tem responsabilidade única, entrada e saída bem definidas, e pode ser substituído ou escalado de forma independente.

---

## Arquitetura — Pipeline de 7 Agentes

```
  Google Maps
      │
      ▼
  ╔═══════════╗   Busca empresas por setor e cidade
  ║ AGENTE 1  ║   Filtra por avaliações e potencial de venda
  ║ Prospector║   Score de prioridade com IA
  ╚═════╤═════╝
        │ leads.json
        ▼
  ╔═══════════╗   Analisa o negócio com Claude AI
  ║ AGENTE 2  ║   Identifica pontos fracos e oportunidades
  ║Diagnóstico║   Calcula score 0–100
  ╚═════╤═════╝
        │ diagnosticos.json
        ▼
  ╔═══════════╗   Gera landing page HTML personalizada
  ║ AGENTE 3  ║   baseada no diagnóstico do negócio
  ║  Builder  ║   Deploy automático via Surge.sh
  ╚═════╤═════╝
        │ pages/{slug}/index.html
        ▼
  ╔═══════════╗   Captura vídeo mobile da landing page
  ║ AGENTE 4  ║   via Puppeteer headless
  ║   Vídeo   ║
  ╚═════╤═════╝
        │
        ▼
  ╔═══════════╗   Gera mensagem de prospecção personalizada
  ║ AGENTE 5  ║   referenciando diagnóstico e landing page
  ║   Canal   ║   adapta tom por setor
  ╚═════╤═════╝
        │ mensagens.json
        ▼
  ╔═══════════╗   Revisa com IA — score de qualidade
  ║ AGENTE 6  ║   Remove padrões genéricos de IA
  ║  Revisor  ║   Reescreve se score < 80
  ╚═════╤═════╝
        │ mensagem aprovada
        ▼
  ┌──────────────────────────────────────┐
  │         PAINEL DE APROVAÇÃO          │  ← único passo manual
  │  Preview da landing page gerada      │
  │  Score de qualidade · histórico      │
  └─────────────────┬────────────────────┘
                    │ aprovado
                    ▼
  ╔═══════════╗   Envia via WhatsApp ou Gmail
  ║ AGENTE 7  ║   Monitora respostas 24/7
  ║  Handler  ║   Responde leads com IA automaticamente
  ╚═══════════╝
```

---

## Decisões de Design

**Por que 7 agentes separados e não um único orquestrador?** Cada agente tem seu próprio ciclo de falha, latência e estado. Separar permite reiniciar um estágio individualmente, monitorar métricas por etapa e substituir a implementação de um agente sem tocar nos outros.

**Por que PM2 em vez de um job scheduler?** O sistema usa `node-cron` para o agendamento diário + PM2 para auto-restart e boot na inicialização do servidor. A combinação elimina a necessidade de um servidor de filas externo para um único pipeline.

**Human-in-the-loop deliberado**: o painel de aprovação não é um workaround — é parte da arquitetura. Automação sem revisão gera spam. A aprovação manual antes do envio é a única etapa não automatizável com segurança neste domínio.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Linguagem | TypeScript / Node.js 18+ |
| IA | Anthropic Claude API |
| Prospecção | Google Places API |
| WhatsApp | whatsapp-web.js |
| Email | Gmail API (OAuth 2.0) |
| Landing Pages | HTML gerado por IA + Surge.sh |
| Screenshots/Vídeo | Puppeteer |
| Servidor | Express.js |
| Processos | PM2 (auto-restart + boot) |
| Agendamento | node-cron |

---

## Estrutura

```
sales-automation/
├── agents/
│   ├── agent1_prospector.ts    # Google Maps → leads
│   ├── agent2_diagnostico.ts   # Claude AI → diagnóstico + score
│   ├── agent3_builder.ts       # HTML personalizado + deploy
│   ├── agent4_video.ts         # captura mobile headless
│   ├── agent5_canal.ts         # mensagem de prospecção
│   ├── agent6_revisor.ts       # revisão de qualidade com IA
│   └── agent7_handler.ts       # envio + respostas automáticas
├── api/
│   └── server.ts               # Express API — painel de aprovação
├── aprovacao.html              # painel web
├── orchestrator.ts             # cron diário — coordena os agentes
└── ecosystem.config.js         # PM2 config
```

---

## Instalação

```bash
git clone https://github.com/victor-germano/agente-de-automacao.git
cd agente-de-automacao
npm install
cp .env.example .env   # preencher as chaves
```

**Configurar Gmail OAuth:**
```bash
npx ts-node scripts/gerar-oauth.ts
```

**Iniciar com PM2:**
```bash
pm2 start ecosystem.config.js
pm2 save
# painel: http://localhost:3000
```

**Variáveis necessárias:**
```env
ANTHROPIC_API_KEY=
GOOGLE_PLACES_API_KEY=
OWNER_WHATSAPP=5511999999999
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REFRESH_TOKEN=
```

---

## Monitorar

```bash
pm2 status       # processos rodando
pm2 logs         # logs em tempo real
pm2 monit        # painel interativo
```

---

## Licença

MIT
