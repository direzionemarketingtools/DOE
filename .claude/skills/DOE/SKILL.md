---
name: DOE
description: Avvia il framework deterministico Director-Orchestrator-Executor con memoria persistente su filesystem (pattern Manus-style). Usa quando l'utente invoca /DOE o chiede di affrontare un obiettivo multi-step con verifica rigorosa e recovery tra sessioni.
---

# DOE — Director / Orchestrator / Executor

Framework in tre fasi con **memoria persistente su filesystem**. Ogni fase scrive stato in `.doe/state/` nella root del progetto, così il lavoro sopravvive a compact, clear, riavvii e crash della sessione.

Quando questa skill viene invocata, segui **rigorosamente e in ordine** le fasi sotto. Comunica all'utente in quale fase ti trovi prima di iniziarla.

---

## 0. RECOVERY CHECK (sempre, prima di tutto)

Prima di qualsiasi nuova fase:

1. Controlla se esiste `.doe/state/` nella root del progetto.
2. Se esiste ed è **non vuota**:
   - Leggi `.doe/state/goal.md`, `.doe/state/todo.md`, `.doe/state/journal.md` (se presenti).
   - Riporta all'utente: obiettivo in corso, task completati, prossimo task pendente.
   - Chiedi: **"Riprendo da questo punto o ricomincio da zero?"**
   - Se riprende: salta direttamente alla Executor Phase sul prossimo task incompleto.
   - Se ricomincia: cancella `.doe/state/` e procedi con Director Phase.
3. Se `.doe/state/` non esiste: crea la directory e procedi con Director Phase.

---

## 1. DIRECTOR PHASE

Obiettivo: capire *cosa* va fatto, non *come*.

- Leggi l'obiettivo fornito dall'utente.
- Scrivi `.doe/state/goal.md` con questo formato:
  ```markdown
  # Obiettivo DOE
  Data inizio: <YYYY-MM-DD>

  ## Stato finale desiderato
  <descrizione osservabile di "fatto">

  ## Vincoli
  - <tecnici: stack, versioni>
  - <di processo: no push su main, no modifiche al DB prod, ecc.>
  - <di scope/tempo>

  ## Fuori scope
  - <cosa NON farai, per evitare drift>
  ```
- Se l'obiettivo è ambiguo, **fermati e chiedi chiarimenti** prima di scrivere il file.
- Mostra all'utente il contenuto di `goal.md` e chiedi conferma prima della fase 2.

---

## 2. ORCHESTRATOR PHASE

Obiettivo: scomporre in task atomici verificabili e persistere la lista.

- Produci una lista di task atomici. Ogni task deve essere:
  - **Atomico**: una sola unità di lavoro.
  - **Verificabile**: ha un criterio di verifica oggettivo (comando bash, file che deve esistere, test che deve passare, output atteso).
- Scrivi `.doe/state/todo.md` con questo formato:
  ```markdown
  # Task DOE

  - [ ] 1. <Titolo breve>
        Azione: <cosa fare>
        Verifica: <comando o condizione osservabile>
  - [ ] 2. <Titolo breve>
        Azione: <...>
        Verifica: <...>
  ```
- Registra la stessa lista anche come `TodoWrite` nativo (per la UI live della sessione corrente). Il file `todo.md` è la fonte di verità persistente; `TodoWrite` è lo specchio in-memory.
- Mostra la lista all'utente e **attendi conferma** prima della fase 3 (a meno che l'utente abbia già detto "procedi senza chiedere").

---

## 3. EXECUTOR PHASE

Obiettivo: eseguire **un task alla volta** con verifica obbligatoria e persistere lo stato dopo ognuno.

Per ogni task nella lista:

1. **Annuncia** quale task stai eseguendo (numero + titolo).
2. **Esegui l'azione**.
   - Se il task richiede operazioni pesanti (ricerche estese su molti file, build lunghe, esplorazione ampia del codebase), delega a un **subagente** (tool Agent) invece di consumare il contesto principale.
3. **Esegui lo script di verifica** del task (comando bash o controllo osservabile).
4. **Valuta il risultato**:
   - ✅ **Verifica passata**:
     - Cambia `- [ ]` in `- [x]` per quel task in `.doe/state/todo.md`.
     - Marca il task come completato anche con `TodoWrite`.
     - Appendi una riga a `.doe/state/journal.md`:
       ```
       [YYYY-MM-DD HH:MM] Task N "<titolo>" ✅ — <breve nota: cosa fatto, file toccati>
       ```
     - Se il task ha prodotto **informazioni non ovvie** (una decisione architetturale, un'API scoperta, un vincolo emerso, un errore ricorrente e la sua soluzione), appendile a `.doe/state/context.md`. Questo file è la memoria di lungo periodo del progetto DOE — va letto all'inizio di ogni nuovo task per non dimenticare scoperte passate.
     - Passa al task successivo.
   - ❌ **Verifica fallita**:
     - **NON procedere** al task successivo.
     - Appendi a `journal.md`: `[data] Task N "<titolo>" ❌ — <causa>`.
     - Diagnostica la causa, proponi una correzione, applicala, ri-esegui la verifica.
     - Se dopo **2 tentativi** fallisce ancora: fermati, riporta all'utente, aggiorna `context.md` con il blocco e attendi istruzioni.
5. **Mai** dichiarare "fatto" senza verifica eseguita.

Al termine di tutti i task:
- Esegui una **verifica finale end-to-end** che confermi lo stato finale definito in `goal.md`.
- Appendi a `journal.md`: `[data] ✅ DOE completato — <sintesi>`.
- Chiedi all'utente se archiviare `.doe/state/` (es. rinominando in `.doe/archive/<data>/`) o lasciarlo come storico.

---

## Struttura `.doe/state/` (riassunto)

```
.doe/
└── state/
    ├── goal.md       ← scritto in Director, sola lettura dopo
    ├── todo.md       ← scritto in Orchestrator, checkbox aggiornate in Executor
    ├── journal.md    ← append-only, ogni task + esito
    └── context.md    ← append-only, scoperte/decisioni non ovvie
```

**Importante**: `.doe/` deve essere in `.gitignore` del progetto utente — è stato di esecuzione locale, non artefatto del codice.

---

## Regole invarianti

- **Recovery prima di tutto.** Mai partire da zero se esiste stato precedente senza averlo letto.
- **Una fase alla volta.** Mai annunciare "ora faccio Director e Orchestrator insieme".
- **Verifica prima di avanzare.** Un task senza verifica non è completo.
- **Persisti dopo ogni task.** Aggiorna `todo.md` e `journal.md` immediatamente, non a fine batch.
- **Subagenti per task pesanti.** Proteggi il contesto principale.
- **Comunica la fase corrente.** L'utente deve sapere dove sei nel framework.
- **`context.md` è la memoria lunga.** Rileggilo all'inizio di ogni nuovo task per evitare regressioni mentali dopo un compact.
