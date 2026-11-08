# 🌅 before-sunrise

**A pre-release audit skill for AI coding agents. Run it before your system sees daylight.**

Works with Claude Code, Codex, Cursor, Antigravity, and any other tool that supports the open [Agent Skills](https://agentskills.io) format.

`before-sunrise` reads your entire codebase and tells you whether it is ready to go live: leaked secrets, missing rate limits, broken access control, and the state machines hiding in your status fields and boolean flags.

It is **read-only**. It reports problems and never touches your code.

Free and open source. Works with any language, framework, or stack: web, API, desktop, and mobile.

[Türkçe](#türkçe)

---

## Why

Most production incidents are not exotic. They look like this:

- A `.env` file that was committed once, deleted, and still lives in git history
- An API key inside a mobile app, a browser bundle, or a desktop binary
- A login endpoint with no rate limit, or a rate limit that treats every user as one IP behind a proxy
- An order that can move from `cancelled` to `shipped` because nothing guards the transition
- A webhook processed twice because it was never made idempotent

Each of these is easy to miss and easy to find, if someone actually looks. This skill looks.

## What it checks

### Phase 1: Security and release readiness

| Area | Examples |
| --- | --- |
| **Secrets** | Environment files in git, full history scan, hardcoded keys, secrets inside anything you ship |
| **Rate limiting** | Coverage of sensitive endpoints, correct limit keys, real client IP behind proxies, brute-force protection |
| **Auth** | Missing authorization, IDOR, tenant isolation, privilege escalation, token lifecycle |
| **Input and integrations** | Injection, XSS, CSRF, mass assignment, file uploads, SSRF, webhook signatures, CORS |
| **Production config** | Debug mode, exposed dev tools and admin panels, HTTPS, cookies, security headers |
| **Dependencies** | Known vulnerabilities, end-of-life runtimes, missing lock files |
| **Operations** | Backups, migrations, queues, scheduled tasks, error reporting |
| **Mobile and desktop** | Secure storage, release build settings, cleartext traffic, TLS validation, permissions, deep links |

Checks that do not apply to your system are marked as such, not silently skipped.

### Phase 2: State machine discovery

Most state machines are never named. They are scattered across status columns, flag combinations, jobs, listeners, and frontend logic. This phase finds them and documents their states, transitions, guards, and side effects, then looks for:

- Invalid or unguarded transitions
- Dead ends and unreachable states
- Race conditions and missing atomicity
- Non-idempotent transitions and duplicate event processing
- Missing retry or recovery paths
- Frontend and backend disagreeing about state

It does not assume everything needs refactoring, and it will not push a state machine library on you without a clear technical reason.

## What you get

One Markdown report, including:

- A release verdict: **ready**, **ready with conditions**, or **not ready**
- Every finding that blocks the release
- Application coverage and known limitations
- Security findings with severity, code references, and impact
- A full state machine inventory, with transition tables and Mermaid diagrams
- A prioritized roadmap, from critical corrections to optional refinements
- Test coverage recommendations

Secret values are always masked in the report, so it stays safe to share with your team.

## Principles

- **Read-only.** No code, schema, configuration, or dependency changes.
- **Evidence only.** Findings reference real code locations. Nothing is invented.
- **Honest about certainty.** Verified findings are separated from hypotheses.
- **Honest about coverage.** Whatever could not be analyzed is stated in the report.
- **Stack-agnostic.** It detects your stack and applies each check using that stack's own mechanisms.

## Installation

### Quick install

One command, for any supported agent. It asks which agents to install to:

```bash
npx skills add bulentsakarya/before-sunrise
```

Add `-g` to install it for all your projects instead of only the current one.

### Manual install

The skill is one folder: [`skills/before-sunrise`](skills/before-sunrise). Clone the repository and copy that folder into your tool's skills directory:

```bash
git clone https://github.com/bulentsakarya/before-sunrise.git
cp -r before-sunrise/skills/before-sunrise <SKILLS_DIRECTORY>/
```

| Tool | This project only | All projects |
| --- | --- | --- |
| **Claude Code** | `.claude/skills/` | `~/.claude/skills/` |
| **Codex** | `.agents/skills/` | `~/.agents/skills/` |
| **Cursor** | `.agents/skills/` or `.cursor/skills/` | `~/.cursor/skills/` |
| **Antigravity** | `.agents/skills/` | `~/.gemini/config/skills/` |

Create the directory if it does not exist, then restart your agent or start a new session so it picks up the skill.

For other tools, check where they load Agent Skills from. The folder itself needs no changes.

### Claude apps

Zip the `skills/before-sunrise` folder, then upload it as a custom skill in your Claude settings. Custom skills require a plan that supports them, with code execution enabled.

## Usage

Open your project and ask:

```text
Run a pre-release audit on this project.
```

```text
Canlıya almadan önce bu sistemi kontrol et.
```

Large codebases take a while. That is expected: the skill is designed not to stop after the first few findings.

## Limitations

- This is an AI-assisted review. It does not replace a professional penetration test or security audit.
- It works from what is in the repository. Server settings, cloud configuration, and real backup status need manual verification, and the report labels them that way.
- An empty report does not prove a system is secure.

## Contributing

Issues and pull requests are welcome, especially checks that are missing for a stack you work with. Please keep the skill stack-agnostic and read-only.

## License

[MIT](LICENSE)

---

## Türkçe

`before-sunrise`, bir sistemi yayına almadan önce çalıştırılan ücretsiz ve açık kaynak bir skill'dir. Claude Code, Codex, Cursor, Antigravity ve Agent Skills formatını destekleyen diğer araçlarla çalışır. Tüm kod tabanını salt okunur olarak inceler, koda dokunmaz.

**İki aşamada çalışır:**

1. **Güvenlik ve yayın hazırlığı:** Git geçmişi dahil sızmış secret'lar, dağıtılan paketlerin içine gömülü API key'ler, rate limit tanımları, yetkilendirme, girdi güvenliği, production ayarları, bağımlılıklar, mobil ve masaüstü kontrolleri.
2. **State machine analizi:** Status alanlarına, flag'lere, job'lara ve event'lere dağılmış tüm durum akışlarını bulur; korumasız geçişleri, çıkmaz akışları, race condition'ları ve tekrar işlenen event'leri raporlar.

**Sonuç:** Yayın kararı (hazır, koşullu hazır, hazır değil), yayını engelleyen bulgular, kod referansları ve önceliklendirilmiş bir yol haritası içeren tek bir Markdown raporu. Secret değerleri raporda maskelenir.

Her dil ve framework ile çalışır.

**Kurulum:**

```bash
npx skills add bulentsakarya/before-sunrise
```

Elle kurulum ve araçlara göre klasörler için yukarıdaki [Installation](#installation) bölümüne bakabilirsin.
