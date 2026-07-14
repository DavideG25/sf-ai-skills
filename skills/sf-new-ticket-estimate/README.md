# sf-new-ticket-estimate

Analisi tecnica verificata + stima di effort + bozza SOW per un ticket Salesforce, **prima** di creare un branch o iniziare l'implementazione.

## Trigger

Usa questa skill quando:
- "fammi una stima per CPQ-2298"
- "quanto costa questo ticket"
- "preparami il SOW per X"
- "preventivo per il ticket X"

## Flusso

| Step | Cosa fa | Si ferma? |
|------|---------|-----------|
| 1. Ticket | Legge `.claude/jira/<TICKET>.md` | No |
| 2. Analisi | Verifica il codice reale (agente `Explore` o diretta) — stessa profondità di `sf-new-ticket` | No |
| 3. Piano | Scrive `.claude/plans/<TICKET>.md` (sezioni separate per MVP se applicabile) | No |
| 4. Stima | Lancia l'agente `sf-estimate-builder` con piano + config di progetto | No |
| 5. Negoziazione | Presenta la tabella, aggiusta a turni con l'utente | **Sì — conversazionale** |
| 6. SOW | Bozza documento SOW su richiesta, salvata con la stima | No |

## Differenza con `sf-new-ticket`

- **Nessun branch creato, nessuna modifica al codice** — è uno strumento di scoping/preventivo.
- L'analisi tecnica è **completa quanto** quella di `sf-new-ticket`, non un pass superficiale: una stima sbagliata per analisi troppo leggera vale meno di zero.
- Se poi si decide di procedere, `sf-new-ticket` riusa il piano già scritto qui (con un controllo di drift sui file referenziati) invece di rifare l'analisi.

## Output

- `.claude/plans/<TICKET>.md` — piano tecnico (stesso file/formato di `sf-new-ticket`)
- `.claude/estimates/<TICKET>.md` — tabella di stima + bozza SOW
- `.claude/sf-estimate-config.md` — config di progetto (template tabella, regole di business), salvata dopo la prima negoziazione

## Note

- Se il piano tecnico risulta poco verificabile (pochi riferimenti a file/righe concrete), la stima va segnalata come indicativa, non affidabile quanto un'analisi verificata a fondo.
- Il primo draft della tabella non è mai definitivo — aspettati un giro di negoziazione con l'utente.
