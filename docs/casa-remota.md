# Base 34 — A Casa Remota: infraestrutura da equipe

**Avatar-Energy · Base 34 · 22 de agosto de 2026**

*Decisão do arquiteto (22/08/2026): a pipeline de desenvolvimento da equipe roda numa instância remota always-on do rakazo, auto-hospedada. Esta base especifica o desenho antes da infraestrutura — o Portão antes do centavo.*

---

## O problema

A equipe precisa conversar com bots persistentes 24/7. A alternativa — rodar o stack na máquina do arquiteto — falha por três motivos verificados: (1) a máquina dorme, e rotinas não disparam; (2) expor a máquina do arquiteto à Internet expõe também suas credenciais GitHub com poder de push em 8 repositórios; (3) o rakazo é beta (nasceu 13/08/2026) e o produto hosted "Rakazo Cloud" não existe — "migrar local↔remoto quando conveniente" não é operação suportada pelo projeto.

A resposta é uma casa só, sempre remota: os bots vivem na nuvem; a máquina do arquiteto é cliente.

## A spec INF

| ID | Requisito | Origem |
|---|---|---| 
| INF-001 | **uma casa só**: uma única instância rakazo, sempre remota e always-on; sem instância local paralela de equipe — a máquina do arquiteto é cliente (browser + Hermes local para trabalho direto no terminal) | decisão |
| INF-002 | **tamanho honesto**: VM 2 vCPU / 2–4 GB RAM / ≥ 20 GB disco (o doc oficial: 2 GB basta para API+worker+Postgres; Docker local de sandbox fica fora — provedores E2B/Daytona assumem os computadores dos bots) | self-host.md |
| INF-003 | **sandbox fora da casa**: computadores dos bots em provedor dedicado (E2B ou Daytona) — a VM roda só API/worker/Postgres; execução isolada, destrutível, nunca na VM da aplicação | self-host.md |
| INF-003b | **credenciais de modelo ficam na casa**: chaves de LLM cifradas no Postgres da VM sob a ENCRYPTION_KEY da instância; exportação por padrão: **nunca** — mesma lei do AVA-006 | AVA-006 |
| INF-004 | **registro fechado**: `SIGNUP_ALLOWLIST` com a equipe do arquiteto; sem registro aberto | decisão |
| INF-005 | **HTTPS público + origem única**: WEB_ORIGIN/BETTER_AUTH_URL/API_URL sob HTTPS com domínio próprio; sem HTTP plano | self-host.md |
| INF-006 | **backup do estado**: dump diário do Postgres + DATA_DIR (homes de bots, perfis de browser) retido 7 dias; restauração testada trimestralmente | operação |
| INF-007 | **custo declarado**: orçamento mensal alvo ≤ US$ 20 (Fly.io 2GB ≈ US$ 10,70; Hetzner CX22 4GB ≈ US$ 4,59 + volumes; E2B/Daytona por uso) — desvio > 20% gera revisão de spec, não silêncio | decisão |
| INF-008 | **atualização controlada**: pin da versão do rakazo; upgrade só após changelog review — beta com breaking changes esperados | self-host.md |
| INF-009 | **Hermes inalterado**: nada nesta base muda o Hermes CLI local do arquiteto — o D1 mensal continua no Hermes até que rotina equivalente exista e seja verificada na casa remota | decisão |
| INF-009b | **divórcio de credenciais**: a casa remota não herda a sessão gh do arquiteto — bots que precisam de GitHub usam token fino (scopo repo de um fork ou deploy key read-only quando possível), nunca o gho_ de pessoa física | decisão |

## O desenho

```
   EQUIPE (browser/mobile, qualquer lugar)
        │ HTTPS
        ▼
┌─────────────────────────────────────────┐
│ A CASA (VM always-on, Fly.io ou Hetzner)│
│  API ─ worker ─ Postgres ─ DATA_DIR     │
│  credenciais de modelo: cifradas aí     │
└────────────────┬────────────────────────┘
                 │ provedor de sandbox
                 ▼
┌─────────────────────────────────────────┐
│ COMPUTERS (E2B/Daytona)                 │
│  browser+shell por bot, efêmeros        │
└──────────────────────────────────────── INF-003

MÁQUINA DO ARQUITETO = cliente (browser) + Hermes local (terminal próprio)
```

## O que não é esta base

- Não é a ponte para "Rakazo Cloud" (não existe); é auto-hospedagem do código aberto
- Não move o trabalho de especificação da casa: specs continuam nascendo nos repositórios, trilíngues, sob SDD
- Não substitui o Hermes do arquiteto (INF-009)

## Papéis

O arquiteto decide provedor e nome de domínio, provisiona credenciais de modelo na casa. As mãos executam: provisionar a VM, subir o stack (guia self-host), configurar allowlist, validar HTTPS, testar backup. Mesmo contrato: rascunho → revisado → verificado.

## Verificação (como esta spec se testa)

| ID | Critério |
|---|---|
| INF-001 | uma única instância listada no provedor; nenhuma segunda instância de equipe |
| INF-002/003 | `docker ps`/systemd na VM mostra só API/worker/Postgres; computers rodam no provedor sandbox |
| INF-005 | curl HTTPS na origem pública: sem downgrade HTTP; certificado válido |
| INF-006 | dump restaurado em ambiente de teste vira instância funcional |
| INF-007 | fatura mensal ≤ US$ 20 ou revisão registrada |

---

*Estado: rascunho — aguarda revisão do arquiteto. Código AGPL-3.0-or-later · Conteúdo CC BY-SA 4.0. Arquitetura e autoria: Cleiton Moura Loura.*
