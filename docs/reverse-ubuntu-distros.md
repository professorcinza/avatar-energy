# Engenharia reversa: Ubuntu Phone e as grandes distribuições → TeiaOS

**Avatar-Energy · Documento base 16 · 22 de agosto de 2026**

*Segundo parâmetro inicial do TeiaOS (o primeiro foi GrapheneOS, base 15): o que o visionário Ubuntu Phone provou antes de morrer, e o que as grandes distribuições vivas provam juntas.*

---

## Parte 1 · Ubuntu Phone — o visionário (2013–2017, e depois)

**O que era**: o sistema convergente da Canonical — anunciado em 2013, o crowdfunding do Ubuntu Edge (US$ 12,8M, recorde histórico) falhou na meta mas provou o apetite; aparelhos reais em 2015 (BQ, Meizu); **abandonado pela Canonical em abril/2017**; sobrevive desde então como **Ubuntu Touch pela comunidade UBports** — hoje em Fairphones, Pixels e outros.

**O que provou (e o TeiaOS herda)**:

| Espec do Ubuntu Touch | Valor para o TeiaOS |
|---|---|
| **Convergência nativa** — dock e vira desktop, janelas reais | a prova original do MOD-011: um dispositivo, dois modos, sem truque |
| **UI declarativa adaptativa** (QML/Unity 8, hoje Lomiri) | a mesma interface escala do bolso à mesa |
| **Atualização OTA atômica** (partições A/B) | sistema nunca quebra no meio da atualização — rollback por definição |
| **Confinamento de apps** (AppArmor) | aplicativo é cidadão confinado, não inquilino com chave da casa |
| **Libertine** — apps de desktop em contêiner no telefone | a ponte entre mundos sem portar nada |
| **Halium** (a solução UBports) | bootar a pilha Linux sobre kernels/HALs Android existentes — **o caminho de transição** |

**Por que morreu (o registro honesto)**: aplicativos que nunca vieram (sem WhatsApp não há usuário; sem usuário não há app), acordos fracos com operadoras/OEMs, e ~US$ 20M+ queimados até o pivô. **A lição de cemitério: o assassino de sistemas operacionais é o ecossistema de apps, não a tecnologia.**

## Parte 2 · As distribuições vivas — o que cada uma prova

| Distribuição | A lição que entra no TeiaOS |
|---|---|
| **Debian** | contrato social e governança comunitária — décadas de democracia funcionando; a forma da fundação neutra (SYS-006) |
| **Arch** | rolling release maduro: sempre o mais recente, estável por disciplina — o "mainstream das atualizações" da base 12 em produção há 20 anos |
| **Fedora** | cadência semestral e *first!* — Wayland, PipeWire, systemd nasceram ali; inovação com porteira |
| **openSUSE** | **openQA**: bateria automatizada de testes que bloqueia regressão antes do usuário — a porteira de SYS-004 de verdade |
| **Ubuntu** | LTS como contrato de confiança (2+5 anos) — o modelo de compromisso público de suporte |
| **NixOS** | builds reproduzíveis, atualização atômica declarativa com rollback — a tecnologia de atualização mais avançada existente |
| **Alpine** | base mínima (musl, pequenez extrema) — menos código = menos ataque (TOS-008) **e menos energia para rodar** |
| **Silverblue/immutable** | base de sistema imutável + apps em camada — o padrão de futuro: OS como imagem, aplicação como contêiner |
| **postmarketOS** | mainline-first em celulares de verdade — o TeiaOS em forma selvagem: Alpine + kernel principal em centenas de aparelhos |

## Parte 3 · Conversão — parâmetros TOS-011 a 020

| ID | Requisito TeiaOS | Origem |
|---|---|---|
| TOS-011 | atualização atômica com rollback automático (A/B ou snapshot) — sistema nunca fica quebrado | Ubuntu Touch / NixOS / openSUSE |
| TOS-012 | base de sistema imutável + aplicativos em contêiner confinado (AppArmor) | Silverblue / UT |
| TOS-013 | rolling com porteira de regressão automatizada estilo openQA — cadência fixa, regressão é bloqueio | Arch / Fedora / openSUSE / SYS-004 |
| TOS-014 | base mínima: nada no sistema que não seja essencial | Alpine |
| TOS-015 | convergência nativa — MOD-011 é também especificação de software | Ubuntu Touch |
| TOS-016 | UI declarativa adaptativa — a mesma interface do bolso à mesa | QML/Lomiri |
| TOS-017 | caminho de transição tipo Halium: bootar em hardware Android existente enquanto o MOD não chega | UBports |
| TOS-018 | governança de contrato social com fundação neutra e eleições | Debian / SYS-006 |
| TOS-019 | pacotes assinados e builds reproduzíveis — cadeia de suprimento verificável | NixOS / GrapheneOS |
| TOS-020 | ponte de apps Android (classe Waydroid) como requisito de sobrevivência, não opção — a lição do cemitério | o erro fatal do Ubuntu Phone |

## O motor gráfico do TeiaOS (decisão, 22/08/2026)

**Mesa + Wayland.** A pilha completa, de baixo para cima:

```
APU (celular / docks / cadeia)          ← APU-004: homogêneas
  DRM/KMS          — kernel mainline: modo, memória, page-flip
  Mesa             — O MOTOR: Vulkan/OpenGL para toda a cadeia
  Wayland          — o protocolo de display; X11 só via XWayland
  wlroots          — fundação de compositor: pequena, padrão, modular (TOS-014)
  shell adaptativa — UI declarativa do bolso à mesa (TOS-016)
```

**Por que Mesa é o motor — e não "um" driver**: Mesa é a implementação aberta de Vulkan/OpenGL para **todos** os silícios da arquitetura: AMD (RADV/RadeonSI), Mali (Panfrost), Adreno (Freedreno/Turnip), Intel (ANV/Iris), renderização por software (LLPipe) e virtualização (virgl/venus). A homogeneidade da hierarquia APU (APU-004 — "mesma pilha em todas as unidades") **é literalmente o modelo Mesa**: um motor, muitos alvos, driver mainline obrigatório (APU-007).

**Por que Wayland + wlroots**: convergência nativa (TOS-015) — saídas múltiplas e densidades diferentes são o caso de uso ordinário do Wayland, não um "modo"; e wlroots entrega a fundação de compositor na medida da base mínima — é a base sob Phosh, Sway e companhia, batalhada em anos de produção.

**A cadeia de APUs sob esse motor**: cada unidade roda seu Mesa local — os displays são dirigidos onde o trabalho renderiza, sem streaming de pixels; e para renderização cruzada entre unidades (app no celular, GPU do dock), o caminho é o mesmo dos contêineres (TOS-012): passthrough de dispositivo, Mesa dentro, nenhum driver novo.

| ID | Requisito | Origem |
|---|---|---|
| TOS-021 | **Mesa é o motor gráfico**: Vulkan/OpenGL para toda a cadeia; nenhum driver fora do mainline entra (APU-007) | decisão do arquiteto |
| TOS-022 | **Wayland + wlroots**: protocolo Wayland nativo; fundação de compositor wlroots; X11 apenas via XWayland | decisão do arquiteto |

*A escolha da shell adaptativa sobre o wlroots (Phosh-classe, Lomiri-classe, ou própria) fica para o arquiteto — a fundação abaixo dela está decidida.*

## A leitura energética

Cada parâmetro tem dimensão energética: base mínima = menos código em execução permanente; imutável = sistema sem *drift*, sem reinstalação de resgate; atualização atômica = **nenhum sistema quebrado pela metade** (a reinstalação é o maior desperdício de software); rollback = anti-desperdício institucionalizado. E a ponte Android (TOS-020) é o que impede o desperdício supremo: **um sistema perfeito que ninguém usa por falta de apps**.

---

*Código AGPL-3.0-or-later · Conteúdo CC BY-SA 4.0. Arquitetura e autoria: Cleiton Moura Loura.*
