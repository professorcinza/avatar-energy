# Teia Engine: o mundo constrói-se enquanto você joga

**Avatar-Energy · Base 35 · 22 de agosto de 2026**

*Decisão do arquiteto: opção A — novo projeto, repositório próprio, specs próprias. Teia Engine é a IA generativa de games do ecossistema: um world model interativo que vai além de game engine. O mundo não é pré-fabricado; nasce do contrato e se constrói em tempo real, localmente, na cadeia de APUs do MOD.*

---

## O que é

**Teia Engine** substitui a game engine tradicional por um **world model interativo**: em vez de desenvolvedores construírem assets, física e lógica por meses, o mundo se **gera enquanto o jogador joga** — visual, física e narrativa juntos, a partir de um contrato markdown ou prompt de texto.

```
GAME ENGINE (Unity/Unreal):
  dev constrói → dev compila → jogador consome (mundo fixo)

TEIA ENGINE (world model):
  jogador escreve contrato → mundo se gera → jogador vive (mundo vivo)
```

## O estado da arte (ago/2026) — reverse spec

| Modelo | Empresa | Real-time | Resolução | Persistência | Aberto |
|---|---|---|---|---|---|
| **[Genie 3](https://deepmind.google/blog/genie-3-a-new-frontier-for-world-models/)** | Google DeepMind | 24 fps | 720p | minutos | ❌ |
| **[MUSE/WHAM](https://www.microsoft.com/en-us/research/blog/introducing-muse-our-first-generative-ai-model-designed-for-gameplay-ideation/)** | Microsoft | ✅ | gameplay | parcial | ✅ pesos |
| **[Oasis 3](https://decart.ai/oasis)** | Decart | ✅ | fotorrealista | horas (dir.) | API |
| **[GameGen-X](https://gamegen-x.github.io/)** | Tencent | parcial | open-world | limitada | ✅ ICLR |
| **[LingBot-World](https://arxiv.org/html/2601.20540v1)** | pesquisa | ✅ | — | limitada | ✅ |
| **[Odyssey](https://odyssey.ml/)** | Odyssey | ✅ | cinema | — | ❌ |

**O gap que ninguém fechou**: todos geram o mundo; **ninguém gera narrativa + mundo + persistência juntos, localmente, aberto**.

## O que Teia Engine tem que ninguém tem

| Diferencial | Por quê ninguém tem |
|---|---|
| **Narrativa LLM integrada** | Genie/MUSE geram visual; não geram história. Teia integra teia-kernel (PET/dialético) como camada narrativa |
| **Contratos markdown como input** | mundo como arquivo versionável, compartilhável pela malha MAL, viajando no microSD |
| **Persistência entre sessões** | a "memória de cristal" do Our-Civilization é a ponte que nenhum world model tem |
| **100% local** | TOS-024: cadeia de APUs, não nuvem. Genie roda em datacenter Google; Teia roda no seu dock |
| **Trilíngue por concepção** | mundo gerável em PT, EN ou ZH desde a primeira linha |

## Arquitetura

```
┌────────────────────────────────────────────────────────┐
│                    TEIA ENGINE                          │
├────────────────────────────────────────────────────────┤
│                                                        │
│  INPUT                                                 │
│  ├── contrato markdown (inkos-worlds)                 │
│  ├── prompt texto                                      │
│  └── seed compartilhada (malha MAL)                   │
│                    ↓                                   │
│  ┌──────────────────┐   ┌────────────────────────┐    │
│  │  WORLD MODEL     │   │  NARRATIVE LLM          │    │
│  │  (visual+físico) │   │  (teia-kernel v22.0)    │    │
│  │                  │   │                         │    │
│  │  gera o mundo    │   │  gera história, NPCs,  │    │
│  │  em tempo real   │   │  quests, diálogos       │    │
│  │  tile-by-tile    │   │  diante das ações       │    │
│  └────────┬─────────┘   └───────────┬─────────────┘    │
│           │        │                 │                  │
│           ▼        ▼                 ▼                  │
│  ┌────────────────────────────────────────────────┐   │
│  │  INTEGRATION LAYER                              │   │
│  │  · física simplificada (colisão, gravidade)    │   │
│  │  · estado persistente (cristal de memória)     │   │
│  │  · sistema de quests/dialog                    │   │
│  │  · rendering (Vulkan via Mesa, TOS-021)       │   │
│  └─────────────────────┬──────────────────────────┘   │
│                        ▼                               │
│  OUTPUT                                                │
│  ├── mundo jogável na tela do Teia Phone               │
│  ├── no dock com a cadeia de APUs (mais mundo/fps)   │
│  └── no gamepad (MOD-013, controles)                  │
│                                                        │
│  TUDO LOCAL · TUDO ABERTO · TUDO TRILÍNGUE            │
│  linguagem: Rust (norma II)                            │
│  licença: AGPL-3.0-or-later                            │
│                                                        │
└────────────────────────────────────────────────────────┘
```

## Especificações

### Input e contratos

| ID | Requisito |
|---|---|
| TE-001 | input primário: **contrato markdown** no formato inkos-worlds — mundo, personagens, regras e narrativa declarados como texto versionável |
| TE-002 | input secundário: **prompt texto livre** — o engine gera um contrato a partir do prompt (LLM local, teia-kernel) |
| TE-003 | **seed determinística**: mesma seed + mesmo contrato = mesmo mundo; seed viaja no microSD, compartilha-se pela malha |
| TE-004 | contratos são trilíngues: o mesmo mundo gerável em PT, EN ou ZH conforme a língua do jogador |

### World model (geração visual e física)

| ID | Requisito |
|---|---|
| TE-005 | geração **tile-by-tile em tempo real**: o mundo se constrói à medida que o jogador explora, não pré-fabricado |
| TE-006 | **estilo inicial: estilizado/low-poly** (estilo Oasis/Minecraft-class) — fotorrealismo é geração futura quando a APU entregar |
| TE-007 | frame rate alvo: **24 fps** no Teia Phone solo, **60 fps** no dock com cadeia de APUs |
| TE-008 | **consistência de mundo**: o mundo mantém geografia, objetos e NPCs coerentes durante a sessão — nada se regenera aleatoriamente |
| TE-009 | física simplificada nativa: colisão, gravidade, interação com objetos — não precisa de engine física externa |
| TE-010 | renderização via **Vulkan/Mesa** (TOS-021) — o mesmo motor gráfico do TeiaOS |

### Narrativa LLM

| ID | Requisito |
|---|---|
| TE-011 | **narrativa gerada diante da ação**: o LLM (teia-kernel) produz diálogo, quests, consequências e eventos a partir das escolhas do jogador em tempo real |
| TE-012 | NPCs com **mentes privadas** (herança Our-Civilization): cada personagem tem conhecimento, motivação e memória próprios |
| TE-013 | sistema de **quests emergentes**: nada scripted; quests nascem da interação entre estado do mundo + ações do jogador + personalidades dos NPCs |
| TE-014 | **diálogo multilíngue**: NPCs falam a língua do jogador (PT/EN/ZH) com sotaques e expressões culturais coerentes |

### Persistência

| ID | Requisito |
|---|---|
| TE-015 | **memória de cristal** (herança OC): 4 níveis de memória — evento, cena, arco, mundo — o mundo lembra o que aconteceu |
| TE-016 | **save = contrato + seed + estado**: o jogo salvo é um arquivo markdown versionável no microSD, compartilhável pela malha |
| TE-017 | **auditor pós-hoc** (herança OC): o engine revisa a própria narrativa após cada sessão para coerência — plot holes são corrigidos |

### Integração com o ecossistema

| ID | Requisito |
|---|---|
| TE-018 | roda em **TeiaOS** (sistema canônico) no Teia Phone e no dock — terminal de primeira classe (TOS-028) para debug e mods |
| TE-019 | consome a **cadeia de APUs** (MOD-012 v4): mais APUs = mais mundo, mais NPCs, mais fps; o avatar aloca (AVA-009) |
| TE-020 | **controles via gamepad MOD-013** (HID padrão) ou tela touch |
| TE-021 | mundos compartilháveis pela **malha MAL**: cada mundo é um arquivo; a teia distribui jogos P2P |
| TE-022 | **open source AGPL-3.0**; Rust (norma II); upstream-first; sem blob |

## As três honestidades

1. **O gap de compute**: Genie 3 roda a 720p/24fps em datacenter Google. A APU chain do MOD não fará fotorrealismo local em 2026-27 — Teia Engine começa **estilizado** (estilo Oasis voxel/Minecraft-class), que é o que a APU consegue gerar em tempo real hoje
2. **A persistência não é resolvida**: nenhum world model atual mantém consistência entre sessões; a memória de cristal do OC é a proposta, mas é **pesquisa aberta**, não tecnologia pronta
3. **Este é o projeto mais ambicioso do ecossistema** — mais que o smartphone, mais que a geladeira. World models interativos em tempo real são a fronteira absoluta da IA. Teia Engine não compete com Unity; compete com **DeepMind**

## Caminho de desenvolvimento

| Fase | O quê | Requisito |
|---|---|---|
| **M0** | protótipo virtual: world model simples rodando em QEMU com renderização software | TE-005/008 |
| **M1** | narrativa LLM integrada: teia-kernel gera quests e diálogo dentro do mundo | TE-011/013 |
| **M2** | persistência: memória de cristal entre sessões | TE-015/016 |
| **M3** | hardware: Teia Phone real com APU chain, 24 fps estilizado | TE-007 |
| **M4** | dock: 60 fps, mundo maior, mais NPCs | TE-019 |
| **M5** | malha: mundos compartilháveis P2P | TE-021 |

## A leitura do avatar

Teia Engine é **a operação distribuir aplicada à imaginação**: mundos que se distribuem pela teia como música se distribui pelo ar. E a conexão energética que fecha o círculo: **o avatar (AVA-009) orquestra as APUs conforme a demanda do mundo** — mais ação = mais APUs acordadas; calmaria = tudo dorme. O jogo que obedece à bateria (INK-003) elevado à escala do mundo inteiro.

---

*Código AGPL-3.0-or-later · Conteúdo CC BY-SA 4.0. Arquitetura e autoria: Cleiton Moura Loura. Estado da arte mapeado em ago/2026; fontes nos links inline.*
