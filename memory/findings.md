# Findings — Project Dashboard

## 2026-09-28 — Ricognizione delle fonti

**Sessioni Claude Code visibili (4)**
| Sessione | Progetto | Stato |
|----------|----------|-------|
| session_01R5TyQbsek7XgVTZfKVr8cn | Translate (System Pilot B.L.A.S.T.) | completato |
| session_01QDWZ7uL2fasZY5yRsMDU2s | J.A.R.V.I.S. | bloccato: attende 2 decisioni dell'utente |
| session_01H8XR5aBaBWVti3NtFsvTEZ | Hermanos Bar & Bistro | completato, in attesa di pubblicazione |
| session_017njacnV7dGQjNme52Suyk6 | Project Dashboard | in corso |

**Repository visibili (2)**: `Translate` (privato), `Jarvis` (pubblico).

## Vincoli scoperti
- **F-001** Progetto ≠ repository: Hermanos vive su un branch di `Translate`.
- **F-002** `Translate` è privato, a differenza di quanto indicato in Discovery ("tutti aperti"). Non cambia nulla per l'agente, che lo legge comunque.
- **F-003** Chat e Progetti di claude.ai non sono leggibili da alcuno strumento → solo input manuale.
- **F-004** Nessun avviso in tempo reale: il controllo più frequente è orario → "subito" = entro 60 minuti.
- **F-005** L'integrazione GitHub non può creare repository (403): il repository va creato a mano dall'utente.
- **F-006** Ogni sessione espone un riepilogo ("cosa serve all'utente", "ultima azione") e i consumi: sono la materia prima per stato, blocchi e consumi.
- **F-007** Il database della pagina ha un limite di 5.000 documenti: le `Run` vanno aggregate (un documento per giorno) o potate, mai una riga per evento senza limite.
- **F-008** Le routine create da questa sessione non possono portarsi dietro il connettore Gmail (parametro `connectors` non disponibile per l'organizzazione). Una routine che avvia una sessione nuova non può quindi mandare mail. Alternative: (a) routine creata dall'utente dall'interfaccia Routine di claude.ai con Gmail attivo; (b) routine che si risveglia in una sessione che ha già Gmail.
- **F-009** Le regole del database consentono le scritture solo al proprietario (verificato: scrittura a livello `interact` respinta).
