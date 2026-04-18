---
name: DOE
description: Avvia il framework deterministico Director-Orchestrator-Executor per task complessi. Usa quando l'utente invoca /DOE o chiede di affrontare un obiettivo multi-step con verifica rigorosa ad ogni passo.
---

# DOE — Director / Orchestrator / Executor

Quando questa skill viene invocata, segui **rigorosamente e in ordine** le tre fasi sotto. Non saltare fasi. Non fondere fasi. Comunica all'utente in quale fase ti trovi prima di iniziarla.

---

## 1. DIRECTOR PHASE

Obiettivo: capire *cosa* va fatto, non *come*.

- Leggi l'obiettivo fornito dall'utente.
- Scrivi in output:
  - **Stato finale desiderato**: una descrizione osservabile di "fatto" (es. "endpoint `/health` risponde 200 in produzione").
  - **Vincoli**: tecnici (stack, versioni), di processo (no push su main, no modifiche al DB), di tempo/scope.
  - **Fuori scope**: cosa NON farai, per evitare drift.
- Se l'obiettivo è ambiguo, **fermati e chiedi chiarimenti** prima di passare alla fase 2.

---

## 2. ORCHESTRATOR PHASE

Obiettivo: scomporre in task atomici verificabili.

- Produci una **lista numerata** di task. Ogni task deve essere:
  - **Atomico**: una sola unità di lavoro.
  - **Verificabile**: ha un criterio di verifica oggettivo (comando bash, file che deve esistere, test che deve passare, output atteso).
- Formato per ogni task:
  ```
  N. [Titolo breve]
     Azione: <cosa fare>
     Verifica: <comando o condizione osservabile>
  ```
- Registra la lista anche come TodoWrite così la progressione è tracciata.
- Mostra la lista all'utente e **attendi conferma** prima di passare alla fase 3 (a meno che l'utente abbia già detto "procedi senza chiedere").

---

## 3. EXECUTOR PHASE

Obiettivo: eseguire **un task alla volta** con verifica obbligatoria.

Per ogni task nella lista:

1. Annuncia quale task stai eseguendo (numero + titolo).
2. Esegui l'azione.
   - Se il task richiede operazioni pesanti (ricerche estese, build lunghe, esplorazione di molti file), delega a un **subagente** (tool Agent) invece di consumare il contesto principale.
3. Esegui lo **script di verifica** del task (comando bash o controllo osservabile).
4. Valuta il risultato:
   - ✅ **Verifica passata** → marca il task come completato (TodoWrite), passa al successivo.
   - ❌ **Verifica fallita** → **NON procedere**. Diagnostica la causa, proponi una correzione, applicala, ri-esegui la verifica. Se dopo 2 tentativi fallisce ancora, fermati e riporta all'utente.
5. Non dichiarare mai "fatto" senza aver eseguito la verifica.

Al termine di tutti i task, esegui una **verifica finale end-to-end** che confermi lo stato finale definito nella Director Phase.

---

## Regole invarianti

- **Una fase alla volta.** Mai annunciare "ora faccio Director e Orchestrator insieme".
- **Verifica prima di avanzare.** Un task senza verifica non è completo.
- **Subagenti per task pesanti.** Proteggi il contesto principale.
- **Comunica la fase corrente.** L'utente deve sempre sapere dove sei nel framework.
