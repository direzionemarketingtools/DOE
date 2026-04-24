# DOE — Director / Orchestrator / Executor

Skill per [Claude Code](https://claude.com/claude-code) che applica un framework deterministico in tre fasi, con **memoria persistente su filesystem** (pattern ispirato a Manus 1.6 Max):

1. **Director** — definisce stato finale e vincoli → li scrive in `.doe/state/goal.md`.
2. **Orchestrator** — scompone in task atomici verificabili → scrive `.doe/state/todo.md`.
3. **Executor** — esegue un task alla volta con verifica obbligatoria, aggiorna checkbox in `todo.md`, registra tutto in `journal.md`, salva scoperte non ovvie in `context.md`. Delega i lavori pesanti a subagenti.

La memoria persistente significa che il tuo lavoro **sopravvive** a `/clear`, `/compact`, riavvii e crash di sessione: al prossimo `/DOE` la skill rileva lo stato esistente e propone di riprendere dal prossimo task incompleto.

Funziona in Claude Code CLI e nelle estensioni IDE (VS Code, JetBrains, Google Antigravity).

---

## Installazione

### Opzione A — skill personale (disponibile in tutti i tuoi progetti)

```bash
git clone https://github.com/direzionemarketingtools/DOE.git /tmp/DOE-repo
mkdir -p ~/.claude/skills/DOE
cp /tmp/DOE-repo/.claude/skills/DOE/SKILL.md ~/.claude/skills/DOE/
rm -rf /tmp/DOE-repo
```

Su Windows (PowerShell):

```powershell
git clone https://github.com/direzionemarketingtools/DOE.git $env:TEMP\DOE-repo
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\DOE" | Out-Null
Copy-Item "$env:TEMP\DOE-repo\.claude\skills\DOE\SKILL.md" "$env:USERPROFILE\.claude\skills\DOE\"
Remove-Item -Recurse -Force "$env:TEMP\DOE-repo"
```

### Opzione B — skill di progetto (solo per una repo specifica)

Dalla root del tuo progetto:

```bash
mkdir -p .claude/skills/DOE
curl -o .claude/skills/DOE/SKILL.md \
  https://raw.githubusercontent.com/direzionemarketingtools/DOE/main/.claude/skills/DOE/SKILL.md
```

### Passo obbligatorio dopo l'installazione

Aggiungi questa riga al `.gitignore` del progetto dove userai DOE:

```gitignore
.doe/
```

`.doe/state/` contiene lo stato di esecuzione locale (obiettivo, todo, journal) — è personale, non va versionato.

---

## Uso

In Claude Code:

```text
/DOE Voglio aggiungere autenticazione JWT al mio backend Express
```

Cosa succede:

1. **Recovery check**: se già esiste `.doe/state/`, ti chiede se riprendere o ricominciare.
2. **Director Phase**: scrive `goal.md` con stato finale + vincoli → ti chiede conferma.
3. **Orchestrator Phase**: scrive `todo.md` con la lista numerata → ti chiede conferma.
4. **Executor Phase**: un task alla volta, con verifica dopo ognuno. Il file `todo.md` si aggiorna in tempo reale (checkbox `- [x]`), `journal.md` registra cosa è stato fatto, `context.md` conserva scoperte importanti per il futuro.

Se interrompi la sessione a metà (o Claude Code crasha, o fai `/clear`), al prossimo `/DOE` ripartirà **esattamente** dal task successivo, avendo riletto `goal.md`, `todo.md` e `context.md`.

---

## Struttura dello stato

```text
tuo-progetto/
└── .doe/
    └── state/
        ├── goal.md       # obiettivo + vincoli (Director)
        ├── todo.md       # lista task con checkbox (Orchestrator + Executor)
        ├── journal.md    # log append-only di ogni task + esito
        └── context.md    # scoperte, decisioni, vincoli emersi
```

---

## Quando usarla

Consigliata per:

- Refactoring multi-file.
- Setup di nuove feature che toccano più parti dello stack.
- Migrazioni.
- Debug complessi dove serve isolare cause una alla volta.
- Sessioni lunghe in cui è probabile un `/compact` o un'interruzione.

Non serve per task banali (un singolo fix, una domanda) — l'overhead delle 3 fasi e dei file di stato non è giustificato.

---

## Licenza

MIT — vedi [LICENSE](LICENSE). Uso libero personale e commerciale.
