# DOE — Director / Orchestrator / Executor

Skill per [Claude Code](https://claude.com/claude-code) che applica un framework deterministico in tre fasi a qualsiasi task complesso:

1. **Director** — definisce stato finale e vincoli.
2. **Orchestrator** — scompone in task atomici verificabili.
3. **Executor** — esegue un task alla volta con verifica obbligatoria, delegando a subagenti i lavori pesanti.

Funziona anche dentro IDE basati su Claude Code (VS Code, Google Antigravity, JetBrains).

---

## Installazione

### Opzione A — skill personale (disponibile in tutti i progetti)

```bash
git clone https://github.com/direzionemarketingtools/DOE.git ~/.claude/skills/DOE-repo
mkdir -p ~/.claude/skills/DOE
cp ~/.claude/skills/DOE-repo/.claude/skills/DOE/SKILL.md ~/.claude/skills/DOE/
```

### Opzione B — skill di progetto (solo per una repo)

Dalla root del tuo progetto:

```bash
mkdir -p .claude/skills/DOE
curl -o .claude/skills/DOE/SKILL.md \
  https://raw.githubusercontent.com/direzionemarketingtools/DOE/main/.claude/skills/DOE/SKILL.md
```

---

## Uso

In Claude Code:

```
/DOE Voglio aggiungere autenticazione JWT al mio backend Express
```

Claude annuncerà la **Director Phase**, poi la **Orchestrator Phase** con la lista numerata di task, e solo dopo la tua conferma passerà alla **Executor Phase**.

---

## Quando usarla

Consigliata per:
- Refactoring multi-file.
- Setup di nuove feature che toccano più parti dello stack.
- Migrazioni.
- Debug complessi dove serve isolare cause una alla volta.

Non serve per task banali (un singolo fix, una domanda) — l'overhead delle 3 fasi non è giustificato.

---

## Licenza

MIT — vedi [LICENSE](LICENSE). Uso libero personale e commerciale.
