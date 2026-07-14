# sf-plan

Analisi tecnica e piano di implementazione per un ticket Salesforce.
Crea il branch, analizza il codice, propone un piano e aspetta approvazione prima di toccare qualsiasi file.

## Trigger

Usa questa skill quando:
- "analizza il ticket X"
- "pianifica lo sviluppo di X"
- "prima di implementare, fammi vedere il piano"
- "sf-plan CPQ-2236"

## Flusso

| Step | Cosa fa | Si ferma? |
|------|---------|-----------|
| 0. Check piano esistente | Se `.claude/plans/<TICKET>.md` esiste già (es. da `sf-new-ticket-estimate`), fa un controllo di drift sui file referenziati invece di rifare l'analisi | No |
| 1. Branch | Crea il feature branch via `/feature` | No |
| 2. Analisi | Esplora il codice rilevante (agente o diretta) — saltato se coperto dallo step 0 | No |
| 3. Piano | Scrive `.claude/plans/<TICKET>.md` e lo mostra — saltato se riusato dallo step 0 | **Sì — aspetta ok** |
| 4. Implementazione | Esegue tutte le modifiche del piano | No |

### Riuso del piano da `sf-new-ticket-estimate`

Se il ticket è già passato da `sf-new-ticket-estimate`, il piano tecnico esiste già. Invece di rifare l'analisi da zero:
1. Prende il timestamp di ultima modifica del file di piano.
2. Controlla `git log --since=<timestamp> -- <ogni file referenziato nel piano>`.
3. Nessun file toccato dopo → riusa il piano com'è. Qualcosa è cambiato → ri-verifica solo le sezioni impattate.

## Analisi: agente o diretta?

- **Agente `Explore`**: se la ricerca coinvolge più di 3 file o pattern non noti
- **Grep/Read diretto**: se la ricerca è semplice e localizzata

## Output

Il piano in `.claude/plans/<TICKET>.md` contiene:
- Cosa cambia e perché
- Lista file da modificare/creare con modifica precisa per ciascuno
- Rischi e dipendenze

## Note

- La skill non modifica mai codice prima dell'approvazione esplicita
- La cartella `.claude/` è in `.gitignore` — i piani non appaiono nelle git changes
- L'implementazione parte solo con conferma esplicita: "vai", "ok", "procedi"
