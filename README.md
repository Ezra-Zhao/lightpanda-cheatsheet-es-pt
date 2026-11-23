# Lightpanda 速查表 | Lightpanda Cheatsheet | Guía rápida de Lightpanda

> 📚 **Colección / Coleção**: esta hoja forma parte de [cheatsheets](https://github.com/Ezra-Zhao/cheatsheets) — 100 hojas de referencia en un solo repositorio. / Esta folha faz parte de [cheatsheets](https://github.com/Ezra-Zhao/cheatsheets) — 100 folhas de referência em um só repositório. ⭐

> 一页看懂 Lightpanda：从零写起的无头浏览器，专为 AI agent 与自动化而生——不是 Chromium 分叉、也不是 WebKit 修补，Zig 编写；官方基准里内存约省 16 倍、跑完同一批页面约快 9 倍。会说 CDP、WebDriver BiDi、MCP 三种「语言」，手头现成的 Puppeteer/Playwright 脚本多半改个连接地址就能跑。

- 上游项目：[lightpanda-io/browser](https://github.com/lightpanda-io/browser)（2026-10-31 实测 39,147 ★ / 1,905 forks，25 个 release，AGPL-3.0；官网 lightpanda.io）
- 平台：Linux / macOS（nightly 二进制）、Docker、Nix；Windows 无原生版，经 WSL2 用 Linux 版
- 本页结构：中文主文 → English → Español → Português

---

Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 官方基准（上游自测口径）

在 AWS EC2 m5.large 上请求 933 个真实页面测得：

| 指标（100 页） | Lightpanda | Headless Chrome | 差距 |
|---|---|---|---|
| 峰值内存 | 123 MB | 2 GB | 约 16 倍更省 |
| 执行时间 | 5 秒 | 46 秒 | 约 9 倍更快 |

这是上游自己的基准、自己的话术口径，胜负手在于「不背图形渲染引擎」：它执行 JavaScript、建 DOM、跑网络请求，但不做像素级排版绘制——你用它抓内容和驱动流程，不用它看页面长什么样。

## 四种用法

| 模式 | 命令 | 干什么 |
|---|---|---|
| 一次性抓取 | `lightpanda fetch` | 打开 URL 直接倒出内容：`--dump html`、`--dump markdown`、`--dump structured_data`（JSON-LD/OpenGraph/卡片/meta 打包成 JSON）、`--dump png` / `--dump pdf`（文字版渲染，见诚实标注） |
| CDP 服务 | `lightpanda serve` | 常驻一个 Chrome DevTools Protocol 服务（默认 9222 口），Puppeteer/Playwright 连上来用；加 `--protocol webdriver` 开 WebDriver BiDi，可与 CDP 同时开 |
| Agent 模式 | `lightpanda agent` | 内置 agent 用自然语言或斜杠命令驱动浏览器（导航、点击、填表、抽数据）；会话可 `/save` 导成 PandaScript——纯 JS＋一组原生浏览器原语，之后 `lightpanda run 脚本.js` 确定性重放，不再消耗模型 token |
| MCP 服务 | `lightpanda mcp` | 原生 MCP（stdio）；加 `--port 9223` 走 HTTP，每个连接一份独立会话（页面/cookie/内存互不串），`session_new` / `session_list` / `session_close` 管理会话 |

## 命令速查

抓取（fetch）：

```
./lightpanda fetch --obey-robots --dump markdown https://example.com
./lightpanda fetch --dump html --wait-selector "article" https://example.com
```

等待条件可调：`--wait-until`、`--wait-ms`、`--wait-selector`、`--wait-script`。`--obey-robots` 表示尊重 robots.txt，批量抓取时建议常开。

起服务（serve / mcp）：

```
./lightpanda serve --host 127.0.0.1 --port 9222
./lightpanda serve --port 9222 --protocol webdriver          # BiDi
./lightpanda serve --unix-socket /run/user/1000/lightpanda.sock   # 改走 Unix socket（0600 权限）
./lightpanda mcp --port 9223
```

Puppeteer 接入只需把连接地址指过去（脚本其余部分基本不动）：

```js
import puppeteer from 'puppeteer-core';
const browser = await puppeteer.connect({ browserWSEndpoint: 'ws://127.0.0.1:9222' });
const ctx = await browser.createBrowserContext();
const page = await ctx.newPage();
await page.goto('https://example.com', { waitUntil: 'networkidle0' });
```

Agent 模式：

```
./lightpanda agent                                  # 自动从环境变量找 API key
./lightpanda agent --task "news.ycombinator.com 的头条是什么?"
./lightpanda agent --no-llm                         # 无模型 REPL
./lightpanda agent --provider gemini --task "..."   # 指定供应商
./lightpanda agent --list-models
./lightpanda run session.js                         # 重放已保存的 PandaScript
```

支持的模型来源：Anthropic、OpenAI、Gemini、Vertex AI、Mistral、Hugging Face、Vercel AI Gateway、OpenRouter、任意 OpenAI 兼容端点（`OPENAI_BASE_URL`），以及 Ollama/llama.cpp 等本地模型。

杂项：`./lightpanda version` 验二进制；`lightpanda completion <bash|fish|zsh>` 生成 shell 补全；默认开遥测，`LIGHTPANDA_DISABLE_TELEMETRY=true` 关闭。

## 安装一行版

| 方式 | 命令 |
|---|---|
| Homebrew（nightly） | `brew install lightpanda-io/browser/lightpanda` |
| Arch（AUR） | `yay -S lightpanda-nightly-bin` |
| 直接下载 nightly（Linux x86_64） | `curl -L -o lightpanda https://github.com/lightpanda-io/browser/releases/download/nightly/lightpanda-x86_64-linux && chmod a+x ./lightpanda`（macOS 与 aarch64 有对应文件名） |
| Docker | `docker run -d --name lightpanda -p 127.0.0.1:9222:9222 lightpanda/browser:latest`（官方镜像，amd64/arm64） |
| Nix | `nix run github:lightpanda-io/browser -- --help`，或把该仓加为 flake input |
| 源码构建 | 需 Zig 0.17.0＋v8/libcurl/html5ever 等依赖，`make build`——只是用浏览器的话没必要走这条 |

## 诚实标注区

- **没有图形渲染引擎**：`--dump png` / `--dump pdf` 出的是文字版渲染，不是页面截图。要像素级截图、视觉回归比对，别指望它，老实用真 Chromium 系。
- **Web 兼容还在追赶**：它从零实现 Web 平台，官方每日公布 Web Platform Tests 结果（perf.lightpanda.io）；复杂站点的边缘 API 可能缺失，关键流程先小批量试跑再放量。
- **Linux 二进制挑 libc**：链接 glibc，需要 2.35（x86_64）/2.38（arm64）；Alpine 这类 musl 发行版直接跑不起来（报 `cannot execute: required file not found`），容器里选 `ubuntu:24.04`、`debian:trixie-slim` 这类底。Android/Termux 同理无原生支持。
- **Windows 要绕 WSL2**：没有原生 Windows 二进制；自动化客户端可以跑在宿主机上，WSL 会自动转发 localhost:9222。
- **默认发遥测**：介意就先设 `LIGHTPANDA_DISABLE_TELEMETRY=true` 再跑。
- **许可证是 AGPL-3.0**：自用、内部自动化没问题；要把它嵌进对外服务分发，先把 AGPL 的义务看清楚。
- **发布渠道以 nightly 为主**：迭代非常快，命令面以当日 `--help` 与官方文档为准；本页写的是 2026-10-31 的口径。

---

## English (summary)

Lightpanda (by lightpanda-io, ~39.1k stars as of 2026-10-31, AGPL-3.0) is a headless browser built from scratch in Zig for AI agents and automation — not a Chromium fork, with no graphical rendering engine. Upstream's own benchmark (933 real pages on an EC2 m5.large) claims ~16x less memory (123 MB vs 2 GB peak over 100 pages) and ~9x faster execution (5s vs 46s) than headless Chrome.

Four ways to use it: `lightpanda fetch` dumps a URL as HTML, Markdown, structured data (JSON-LD/OpenGraph/meta as JSON), or text-only PNG/PDF; `lightpanda serve` runs a CDP server on port 9222 (add `--protocol webdriver` for WebDriver BiDi) that existing Puppeteer/Playwright scripts can connect to; `lightpanda agent` drives the browser in plain language and can save sessions as deterministic, token-free PandaScripts replayed with `lightpanda run`; `lightpanda mcp` exposes a native MCP server (stdio, or HTTP on `--port 9223` with isolated per-client sessions). Install via Homebrew, the AUR, nightly binaries, the official Docker image, or Nix.

Honest notes: no pixel rendering, so no real screenshots; web-platform coverage is still catching up (WPT results are published daily); Linux binaries need glibc 2.35/2.38 and won't run on musl distros; Windows goes through WSL2; telemetry is on by default (`LIGHTPANDA_DISABLE_TELEMETRY=true` turns it off).

---

## Español (guía rápida — traducción de estudio, pendiente de revisión por un hablante nativo)

Lightpanda (de lightpanda-io, ~39,1 mil estrellas en octubre de 2026, AGPL-3.0) es un navegador headless escrito desde cero en Zig para agentes de IA y automatización: no es un fork de Chromium y no tiene motor de renderizado gráfico. Según el benchmark del propio proyecto (933 páginas reales en un EC2 m5.large), usa unas 16 veces menos memoria (123 MB frente a 2 GB en 100 páginas) y termina unas 9 veces más rápido (5 s frente a 46 s) que Chrome headless.

**Cuatro modos de uso:**

- **`lightpanda fetch`**: vuelca una URL en HTML, Markdown, datos estructurados (`--dump structured_data`: JSON-LD, OpenGraph y meta en JSON) o PNG/PDF solo de texto. Ajustes de espera: `--wait-until`, `--wait-ms`, `--wait-selector`, `--wait-script`; `--obey-robots` respeta robots.txt.
- **`lightpanda serve`**: servidor CDP en el puerto 9222 (con `--protocol webdriver` también WebDriver BiDi; `--unix-socket` como alternativa a TCP). Tus scripts de Puppeteer/Playwright se conectan cambiando solo el endpoint: `puppeteer.connect({ browserWSEndpoint: 'ws://127.0.0.1:9222' })`.
- **`lightpanda agent`**: un agente integrado conduce el navegador con lenguaje natural; la sesión se puede guardar con `/save` como PandaScript y repetirse sin gastar tokens con `lightpanda run script.js`. `--no-llm` abre un REPL sin modelo; proveedores: Anthropic, OpenAI, Gemini, Vertex, Mistral, OpenRouter, endpoints compatibles con OpenAI y modelos locales (Ollama).
- **`lightpanda mcp`**: servidor MCP nativo por stdio, o HTTP con `--port 9223` y sesiones aisladas por cliente.

**Instalación**: `brew install lightpanda-io/browser/lightpanda` (nightly), AUR (`lightpanda-nightly-bin`), binario nightly directo desde GitHub Releases, imagen oficial de Docker (`lightpanda/browser:latest`) o Nix. Sin binario nativo para Windows: se usa dentro de WSL2.

**Notas honestas**: sin renderizado de píxeles no hay capturas reales (el PNG/PDF es solo texto); la compatibilidad web aún va por detrás (publican a diario sus resultados de Web Platform Tests); los binarios de Linux exigen glibc 2.35/2.38 y no corren en distros musl como Alpine; la telemetría viene activada por defecto (se apaga con `LIGHTPANDA_DISABLE_TELEMETRY=true`); y la licencia AGPL-3.0 hay que mirarla con calma si piensas ofrecerlo como servicio.

---

## Português (guia rápido — tradução de estudo, aguardando revisão de um falante nativo)

O Lightpanda (do lightpanda-io, ~39,1 mil estrelas em outubro de 2026, AGPL-3.0) é um navegador headless escrito do zero em Zig para agentes de IA e automação: não é um fork do Chromium e não tem motor de renderização gráfica. Segundo o benchmark do próprio projeto (933 páginas reais em um EC2 m5.large), usa cerca de 16 vezes menos memória (123 MB contra 2 GB em 100 páginas) e termina cerca de 9 vezes mais rápido (5 s contra 46 s) que o Chrome headless.

**Quatro modos de uso:**

- **`lightpanda fetch`**: despeja uma URL em HTML, Markdown, dados estruturados (`--dump structured_data`: JSON-LD, OpenGraph e meta em JSON) ou PNG/PDF só de texto. Ajustes de espera: `--wait-until`, `--wait-ms`, `--wait-selector`, `--wait-script`; `--obey-robots` respeita o robots.txt.
- **`lightpanda serve`**: servidor CDP na porta 9222 (com `--protocol webdriver` também WebDriver BiDi; `--unix-socket` como alternativa ao TCP). Seus scripts de Puppeteer/Playwright conectam mudando só o endpoint: `puppeteer.connect({ browserWSEndpoint: 'ws://127.0.0.1:9222' })`.
- **`lightpanda agent`**: um agente embutido conduz o navegador em linguagem natural; a sessão pode ser salva com `/save` como PandaScript e repetida sem gastar tokens com `lightpanda run script.js`. `--no-llm` abre um REPL sem modelo; provedores: Anthropic, OpenAI, Gemini, Vertex, Mistral, OpenRouter, endpoints compatíveis com OpenAI e modelos locais (Ollama).
- **`lightpanda mcp`**: servidor MCP nativo por stdio, ou HTTP com `--port 9223` e sessões isoladas por cliente.

**Instalação**: `brew install lightpanda-io/browser/lightpanda` (nightly), AUR (`lightpanda-nightly-bin`), binário nightly direto do GitHub Releases, imagem oficial do Docker (`lightpanda/browser:latest`) ou Nix. Sem binário nativo para Windows: usa-se dentro do WSL2.

**Notas honestas**: sem renderização de pixels não há capturas reais (o PNG/PDF é só texto); a compatibilidade web ainda está atrás (publicam diariamente os resultados do Web Platform Tests); os binários de Linux exigem glibc 2.35/2.38 e não rodam em distros musl como o Alpine; a telemetria vem ligada por padrão (desliga com `LIGHTPANDA_DISABLE_TELEMETRY=true`); e a licença AGPL-3.0 precisa ser lida com calma se a ideia for oferecê-lo como serviço.

---

## 来源与声明

- 灵感来源 / Source / Fuente / Fonte: [lightpanda-io/browser](https://github.com/lightpanda-io/browser)（官网 lightpanda.io）
- 本仓是个人学习整理：内容用我自己的话总结自上游公开文档，不搬运原文；基准数字为上游自测口径，命令与参数以 2026-10-31 的上游 README 为准，上游迭代很快，请以官方文档为最终依据。
- 西语、葡语部分为学习译稿，待母语校对。Las secciones en español y portugués son traducciones de estudio pendientes de revisión nativa.
- License: MIT © 2026 Guangyi Zhao
