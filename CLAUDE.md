# CLAUDE.md — Controle de Água 💧

Guia para o Claude Code (e para humanos) trabalharem neste projeto.

## O que é

App web de **acompanhamento de consumo diário de água**, em português (pt-BR).
Meta diária de **2 L**, dividida em **Manhã (1 L)** e **Tarde (1 L)**, contadas em
**copos de 250 ml** (4 copos por período). Inclui um lembrete de **chimarrão**
(beber 500 ml antes) e um **histórico semanal** em barras.

Publicado como **GitHub Pages**: https://wagliardi.github.io/cc-agua
Repositório: conta **Wagliardi**, repo `cc-agua`.

## Escopo atual: apenas HTML

➡️ **Daqui pra frente trabalhamos somente com o app web estático (`index.html`).**

Toda a parte de **Tauri / desktop / mobile** está **descontinuada** e seus
artefatos foram movidos para a pasta **`legado/`**:

```
legado/
  Controle de Água_1.0.0_x64-setup.exe   # instalador Tauri (Windows)
  controle-agua-portatil.exe             # versão portátil Tauri
  controle-agua-portatil.zip
  controle-agua.apk                      # build Android
```

> **Ignore a pasta `legado/`.** São apenas builds antigos guardados para
> referência — não são código-fonte e não devem ser editados, atualizados nem
> reempacotados. Não considere mais Tauri/`file://`/empacotamento nas decisões.

## Regra de ouro (NÃO QUEBRAR)

⚠️ **Tudo deve ser 100% estático e nativo de HTML/CSS/JS.**

- **Sem build, sem bundler, sem framework, sem npm em runtime.** O arquivo é
  servido como está pelo GitHub Pages.
- **Sem dependências de CDN externas.** Tudo que for usado (ex.: ícones Tabler)
  deve estar versionado localmente em `./assets/`. Nada de buscar JS/CSS de
  servidores externos — o app precisa funcionar offline.
- **Sem backend.** Toda persistência é via `localStorage` no próprio navegador.
- **Caminhos sempre relativos** (`./...`), nunca absolutos com domínio, porque o
  Pages serve sob o subcaminho `/cc-agua/`.

Se algo só funciona com servidor/ferramenta de build, **não pertence a este projeto.**

## Estrutura esperada do repositório `cc-agua`

```
index.html                  # App inteiro: HTML + <style> + <script> inline
manifest.webmanifest        # Manifesto PWA
sw.js                       # Service worker (cache offline)
icon-192.png                # Ícone PWA / apple-touch-icon
assets/tabler/
  tabler-icons.min.css      # Ícones Tabler (local, sem CDN)
  fonts/...                 # Fontes dos ícones
```

## Arquitetura do `index.html`

Tudo num único arquivo, sem frameworks:

- **CSS** num `<style>` inline. Paleta: verde `#1D9E75` (água/meta), âmbar
  `#BA7517` (manhã/chimarrão), fundo `#f4f6f0`. Layout mobile-first com
  `max-width: 420px`.
- **JS** num `<script>` inline. Constantes principais:
  - `CUP = 250` (ml por copo), `GOAL = 2000` (meta em ml)
  - Chaves de `localStorage`: `agua_date`, `agua_state`, `agua_history`
- **Estado** (`state`): `{ manha, tarde, chimarrao }`. Total =
  `(manha + tarde) * 250 + (chimarrao ? 500 : 0)`.
- **Funções-chave:**
  - `dateKey()/todayStr()` — usam data **local** (não UTC) de propósito, para o
    dia não virar às 21h em UTC-3.
  - `loadState()` — ao detectar novo dia, **arquiva** o total do dia anterior em
    `agua_history` e zera o estado.
  - `saveState()` — persiste estado + atualiza histórico + mostra badge "salvo".
  - `buildCups()` — gera os 4 botões de copo por período.
  - `renderWeek()` — desenha o histórico de Seg→Dom (semana começa na segunda).
  - `refresh()` — recalcula total, barra de progresso, "parabéns" e semana.
- **Service worker** só é registrado quando `location.protocol` é http/https.

## Convenções ao editar

- Mantenha **HTML, CSS e JS no mesmo `index.html`** — é proposital.
- Texto sempre em **pt-BR**; números no formato brasileiro (vírgula decimal,
  ex.: `1,5 L` via `.toFixed(1).replace('.', ',')`).
- Ícones: use classes Tabler já disponíveis localmente (`<i class="ti ti-...">`).
  Não adicione bibliotecas de ícones novas via CDN.
- Ao mexer no schema do `localStorage`, lembre da **migração**: usuários têm
  dados antigos salvos; não quebre `agua_state`/`agua_history`.
- Se alterar o cache offline, **atualize a versão do cache no `sw.js`** para os
  usuários receberem a nova versão.

## Como testar localmente

Como é estático, basta abrir num servidor simples (necessário p/ service worker
e manifesto funcionarem — `file://` não registra SW):

```bash
# na raiz do repositório cc-agua
python -m http.server 8000
# abrir http://localhost:8000
```

Abrir direto o `index.html` (`file://`) também funciona para a lógica de copos e
histórico — apenas o PWA/offline não ativa.

## Deploy

Push para a branch publicada pelo GitHub Pages do repo `cc-agua`. Não há etapa de
build: o que está no repositório é o que vai para o ar em
https://wagliardi.github.io/cc-agua

## Legado (Tauri / desktop / mobile) — descontinuado

A versão desktop (Tauri/Windows) e o APK Android **não são mais mantidos**. Os
artefatos ficam em `legado/` apenas como histórico. Não gerar novos pacotes nem
considerar `file://`/Tauri ao editar o app.
