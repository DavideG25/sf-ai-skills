---
name: sf-new-ticket
description: >
  Workflow completo per avviare un nuovo ticket Salesforce: crea il branch feature da release,
  analizza il codice esistente rilevante (pattern da estendere, file coinvolti, dipendenze),
  scrive un piano tecnico dettagliato in .claude/plans/<TICKET>.md e si ferma per conferma
  esplicita prima di toccare qualsiasi file di codice. Implementa solo dopo un "vai"/"ok"/"procedi".
  Usa questa skill ogni volta che l'utente fornisce un numero di ticket Jira (es. CPQ-2236,
  TGL-895, SFC-1234) insieme alla descrizione di cosa va sviluppato — anche se non dice
  esplicitamente "crea un piano" o "nuovo ticket". Trigger tipici: "CPQ-2236: aggiungi il campo X",
  "devo lavorare sul ticket TGL-895 che fa Y", "nuovo ticket ABC-123, mi serve Z",
  "analizza e pianifica questo ticket".
---

# sf-plan
Workflow: analisi tecnica + piano di implementazione per un ticket Salesforce.

## Quando usare questa skill
Quando l'utente fornisce un numero di ticket Jira (es. CPQ-2236, TGL-895) e descrive cosa deve essere sviluppato.

## Comportamento

### 0. Controlla se esiste già un piano
Prima di tutto, controlla se `.claude/plans/<TICKET>.md` esiste già (es. da una precedente run di `sf-new-ticket-estimate`).

- **Se non esiste**: procedi normalmente dallo step 1, comportamento invariato.
- **Se esiste**: non fidartene ciecamente né rifare tutta l'analisi da capo — fai un controllo di drift mirato:
  1. Prendi il timestamp di ultima modifica del file di piano.
  2. Per ogni file/path citato nel piano, esegui `git log --since=<quel timestamp> -- <file>` per vedere se ci sono commit successivi su quei file specifici.
  3. **Nessun file referenziato ha commit successivi** → il piano è presumibilmente ancora valido: salta lo step 2 (analisi) e vai direttamente allo step 1 (branch), poi step 4 (presenta il piano già esistente per conferma).
  4. **Qualcosa è cambiato** → segnala all'utente quali sezioni del piano potrebbero essere superate (solo quelle legate ai file con commit successivi) e ri-verifica mirata solo quelle parti, non l'intero piano da zero.

### 1. Crea il branch
Esegui la skill `/feature $ARGUMENTS` per creare il branch feature partendo da release.

### 2. Analisi del codice
Salta questo step se già coperto dallo step 0 (piano esistente e senza drift). Altrimenti, analizza il codice esistente rilevante per il ticket.

**Se la ricerca coinvolge più di 3 file o pattern non noti**: lancia un agente `Explore` con un prompt dettagliato che descrive cosa cercare (classi, trigger, metadata, campi, pattern esistenti da estendere).

**Se la ricerca è semplice e localizzata**: usa direttamente Grep/Read senza agente.

In entrambi i casi, identifica:
- I file coinvolti con path e righe esatte
- Il pattern già in uso (per replicarlo o estenderlo)
- Le dipendenze tra i file

### 3. Scrivi il piano tecnico
Salta questo step se il piano esistente è stato riusato senza drift (step 0). Altrimenti, crea/aggiorna il file `.claude/plans/<TICKET>.md` con:
- Descrizione sintetica di cosa cambia e perché
- Lista dei file da modificare/creare, con la modifica precisa per ciascuno
- Eventuali rischi o dipendenze da verificare

### 4. Presentati e fermati
Mostra il piano all'utente in chat.  
**Non toccare nessun file di codice.** Aspetta conferma esplicita prima di procedere.

### 5. Implementazione (solo dopo ok esplicito)
Quando l'utente dice "vai", "ok", "procedi" o equivalente, implementa tutte le modifiche descritte nel piano.
