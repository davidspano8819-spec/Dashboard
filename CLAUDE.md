# CLAUDE.md — Costituzione del Progetto «Project Dashboard»

> Documento normativo. Precede il codice e lo vincola.
> Protocollo: **B.L.A.S.T.** · Costruzione: **A.N.T.**
> Principio guida: **Affidabilità prima della velocità. Mai indovinare la
> business logic.**

---

## 0. Stato del Progetto

| Campo | Valore |
|-------|--------|
| Fase corrente | **L — Link** — verifica dei collegamenti |
| Gate `/execution/` | 🔓 **SBLOCCATO** (Blueprint approvato) |
| Schema Dati | ✅ definito e confermato (§1) |
| Blueprint approvato | ✅ sì, 2026-09-28 |
| Discovery | ✅ 10/10 risposte registrate (2026-09-28) |
| Ultimo aggiornamento | 2026-09-28 |

**Regola di sblocco**: il gate su `/execution/` si apre solo quando lo
Schema Dati (§1) è confermato e il Blueprint in `memory/task_plan.md` è
approvato dall'utente.

---

## 1. Schema Dati (Regola Data-First)

> Contratto vincolante. Se lo schema cambia, si aggiorna qui **prima** di
> toccare `/execution/`.

### 1.0 Concetto chiave: progetto ≠ repository

Un **progetto** è un'unità di lavoro dell'utente. Può avere zero, uno o più
repository e una o più sessioni Claude Code (es. «Hermanos» vive su un branch
di `Translate` e non ha un repository proprio). La Dashboard ragiona **per
progetto**; repository e sessioni sono solo *fonti* collegate.

### 1.1 `Project` — collezione `projects`

```json
{
  "id": "jarvis",
  "name": "J.A.R.V.I.S. assistente vocale personale",
  "status": "blocked",
  "status_source": "auto",
  "deadline": "2026-10-15",
  "deadline_source": "manual",
  "sources": {
    "sessions": ["session_01QDWZ7uL2fasZY5yRsMDU2s"],
    "repos": ["davidspano8819-spec/Jarvis"],
    "branches": ["claude/happy-edison-iyrndl"]
  },
  "current_state": "Fase 1 pubblicata su main; Fase 2 ferma su 2 decisioni.",
  "phase_summary": "Setup → Fase 1 (voce + persona) completata il 2026-09-27.",
  "next_steps": ["Testare la Fase 1 in locale", "Scegliere il server per la Fase 2"],
  "risks": ["Costo server ~5 €/mese se si sceglie il cloud"],
  "blockers": ["Decisione server Fase 2"],
  "last_activity_at": "2026-09-27T11:10:56Z",
  "days_idle": 1,
  "usage": { "cost_usd_to_date": 2.94, "sessions_count": 1 },
  "manual_notes": "",
  "opened_at": "2026-09-27T10:58:18Z",
  "closed_at": null
}
```

**Valori di `status`**: `active | paused | blocked | done | closed`.
- `done` = lavoro finito, progetto ancora seguito (es. in attesa di pubblicazione).
- `closed` = chiuso dall'utente → esce dalla Dashboard (vedi §1.6).

**Regola di precedenza (D-006)**: ogni campo modificabile a mano (`status`,
`deadline`, `manual_notes`) porta un `*_source: auto | manual`. Un valore
`manual` **non viene mai sovrascritto** dall'agente; l'agente può solo
proporre una modifica nella mail.

### 1.2 `Milestone` (tappa) — collezione `milestones`

```json
{
  "id": "jarvis__2026-09-27__release-phase1",
  "project_id": "jarvis",
  "date": "2026-09-27",
  "kind": "release",
  "title": "Fase 1 pubblicata",
  "detail": "commit 2e9ec31 su main",
  "source": "session_01QDWZ7uL2fasZY5yRsMDU2s"
}
```

`kind`: `decision | blocker | release | risk | next_step | phase_start | phase_end | manual`.
`id` deterministico → la stessa tappa non viene mai registrata due volte.

### 1.3 `Alert` — collezione `alerts`

```json
{
  "id": "jarvis__awaiting_input__2026-09-28",
  "project_id": "jarvis",
  "rule": "awaiting_input",
  "severity": "urgent",
  "message": "La sessione J.A.R.V.I.S. attende una tua risposta da 26 ore.",
  "first_seen_at": "2026-09-28T12:00:00Z",
  "emailed_at": "2026-09-28T12:00:40Z",
  "resolved_at": null
}
```

### 1.4 `Run` — collezione `runs` (registro di ogni esecuzione)

```json
{
  "id": "2026-09-29T05:50:00Z__daily",
  "kind": "daily | hourly",
  "status": "success | partial | failed",
  "projects_scanned": 4,
  "summaries_regenerated": true,
  "alerts_emailed": 1,
  "issues_opened": ["davidspano8819-spec/Dashboard#3"],
  "email_sent": true,
  "errors": []
}
```

### 1.5 Regole deterministiche

| Regola | Definizione |
|--------|-------------|
| **Fermo** (`stale`) | `status ∈ {active, blocked}` e `days_idle ≥ 7` |
| **Urgente: scadenza** | `deadline` entro 48 ore e `status ∉ {done, closed}` |
| **Urgente: attesa risposta** | sessione collegata in stato "attende input" da > 24 ore |
| **Urgente: agente guasto** | una `Run` con `status = failed` |
| **Anti-spam** | lo stesso `Alert.id` (progetto + regola + giorno) si invia una volta sola |
| **Issue** | una sola issue aperta per coppia progetto + causa; se esiste già, si commenta invece di duplicare |

### 1.6 Conservazione

- Finché il progetto non è `closed`: stato attuale + `phase_summary` + tappe complete.
- Alla chiusura: resta solo una scheda finale (`current_state` + `phase_summary`);
  tappe e alert del progetto vengono eliminati (D-008).

---

## 2. Regole Comportamentali

**Must-do**
- Mail giornaliera con la Dashboard alle **07:50 (Europe/Rome)**, tutti i giorni.
- Mail immediata (entro 60 minuti) su ogni evento urgente (§1.5).
- Segnalare i progetti fermi da ≥ 7 giorni e proporre i prossimi passi.
- Aprire issue per blocchi e prossimi passi concreti.
- Rigenerare i riassunti ogni giorno per i primi 30 giorni, poi ogni lunedì.

**Must-not-do**
- ❌ Sovrascrivere un valore inserito a mano dall'utente.
- ❌ Inventare stato o tappe non presenti nelle fonti: se una fonte non è
  leggibile, lo si dice nella mail.
- ❌ Inviare mail a indirizzi diversi da quello dell'utente.
- ❌ Chiudere issue, fare merge o modificare il codice degli altri progetti.
- ❌ Fallire in silenzio: ogni esecuzione fallita genera un alert.

---

## 3. Invarianti Architetturali

1. **Data-First** — nessuno script prima che lo schema sia definito in §1.
2. **Determinismo** — calcoli (giorni fermo, urgenze, deduplicazione, id)
   vivono in script testabili in `/execution/`. Il modello linguistico
   scrive i riassunti e propone i prossimi passi, non calcola le regole.
3. **SOP prima del codice** — si aggiorna `/architecture/` e poi il codice.
4. **Segreti fuori dal repository** — nessuna credenziale nel codice o nei log.
5. **Intermedi in `/.tmp/`**, effimero e non versionato.
6. **Nessun rilascio senza verifica** — test, screenshot o comando a una riga.
7. **Link rotto = stop** — nessuna logica sopra una fonte non verificata.
8. **Modifiche chirurgiche** — niente astrazioni speculative.
9. **Costo zero** — solo servizi inclusi nell'abbonamento Claude o gratuiti.
10. **Definizione di Completo** — la mail arriva nella casella e la pagina
    web è aggiornata. Un file locale non è una consegna.

---

## 4. Output delle Fasi B.L.A.S.T.

### B — Blueprint (Discovery del 2026-09-28)
| Domanda | Risposta |
|---------|----------|
| Perimetro | Tutti i progetti Claude / Claude Code dell'utente; oggi 4, a regime 10–15 |
| North Star | Ogni mattina: cosa è fermo, cosa fare oggi, a che punto è ogni progetto, quanto si è consumato |
| Fonti | Sessioni Claude Code + repository GitHub collegati + inserimenti manuali |
| Input manuale | Sì: cambio di stato e scadenze dalla pagina web |
| Riassunti | Cronologia per tappe con prossimi passi, rischi, decisioni, blocchi, rilasci |
| Frequenza | Mail ogni giorno 07:50; riassunti giornalieri per 30 giorni, poi ogni lunedì 07:50 |
| Storico | Stato attuale + sintesi fasi, fino alla chiusura del progetto |
| Autonomia | Attiva: apre issue, segnala progetti fermi, propone prossimi passi |
| Notifiche | Mail giornaliera + mail immediata sulle urgenze |
| Consegna | Pagina web pubblicata, privata, ottimizzata per telefono e iPad |
| Vincoli | Costo zero; progetti non riservati; progetto separato |

### L — Link

| Servizio | Probe | Esito |
|----------|-------|-------|
| Sessioni Claude Code (`list_sessions`) | lettura del 2026-09-28 | 🟢 4 sessioni leggibili |
| Repository GitHub (`list_repos`) | lettura del 2026-09-28 | 🟢 2 repository leggibili |
| Repository `davidspano8819-spec/Dashboard` | clone + push | 🟢 creato dall'utente, collegato |
| Gmail (invio) | mail di prova del 2026-09-28 | 🟢 ricevuta dall'utente |
| Pagina web + database | pubblicazione + seed + scrittura negata a livello `interact` | 🟢 https://claude.ai/artifact/M41TgvAL4owkG3sjkHRJqb |
| Esecuzioni programmate | routine singola `trig_014xTu9RT7F49pkg8StZ8sGD` (17:00 UTC) | 🟡 in esecuzione; Gmail non disponibile nelle sessioni nuove (F-008) |
| Chat e Progetti di claude.ai | — | ⛔ non raggiungibili da strumenti: solo input manuale |

### A — Architect (A.N.T.) — proposta

**A — Architecture** (`/architecture/`)
| SOP | Copre |
|-----|-------|
| `SOP-01-raccolta.md` | lettura delle fonti e mappatura sessione/repository → progetto |
| `SOP-02-regole.md` | fermo, urgenze, anti-spam, precedenza manuale |
| `SOP-03-riassunti.md` | formato della cronologia e dei prossimi passi |
| `SOP-04-consegna.md` | mail giornaliera, mail urgente, aggiornamento pagina |
| `SOP-05-azioni.md` | apertura issue e deduplicazione |

**N — Navigation** — un'esecuzione programmata avvia una sessione Claude
Code che legge le fonti, chiama gli script deterministici, scrive nel
database della pagina e invia le mail.

**T — Tools** (`/execution/`) — da definire dopo l'approvazione.

### S — Stylize
- Pagina web: una scheda per progetto, semaforo di stato, cronologia
  espandibile, campi modificabili per stato e scadenza. Tema chiaro/scuro.
- Mail: stesso contenuto in forma compatta, leggibile da telefono.

### T — Trigger
| Esecuzione | Pianificazione (Europe/Rome) | Scopo |
|------------|------------------------------|-------|
| Giornaliera | `50 7 * * *` | raccolta completa, riassunti, mail, issue |
| Oraria | ogni ora | solo controllo urgenze + mail immediata |

---

## 4-bis. Convenzione di Lingua (vincolante)

- **Italiano**: `CLAUDE.md`, SOP, memoria, interazione con l'utente, mail e pagina web.
- **Inglese**: identificatori, nomi di file, commit, commenti di codice, log tecnici.

---

## 5. Struttura File

```
CLAUDE.md          Costituzione + stato
/web/              Pagina web della Dashboard (pubblicata come artifact)
/memory/           task_plan.md · findings.md · progress.md · decisions.md
/architecture/     SOP (il "Come-Fare")
/execution/        Script deterministici (bloccato fino ad approvazione)
/.tmp/             Banco di lavoro temporaneo (non versionato)
```

---

## 6. Log di Manutenzione

| Data | Sintomo | Causa radice | Patch | SOP aggiornata |
|------|---------|--------------|-------|----------------|
| — | — | — | — | — |
