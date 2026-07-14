---
name: sf-new-ticket-estimate
description: >
  Analisi tecnica completa + stima di effort + bozza SOW per un ticket Salesforce, PRIMA di creare
  un branch o iniziare l'implementazione. Legge il ticket da .claude/jira/<TICKET>.md, fa la stessa
  analisi tecnica approfondita di sf-new-ticket (verifica il codice reale, non solo il testo del
  ticket) e scrive il piano in .claude/plans/<TICKET>.md, poi produce una tabella di stima
  (ore/persone/sessioni) e una bozza di documento SOW in .claude/estimates/<TICKET>.md. Nessun
  branch creato, nessuna modifica al codice — è uno strumento di scoping/preventivo, non di
  esecuzione. Usa questa skill quando l'utente chiede una stima, un preventivo, o un SOW per un
  ticket, prima di dare il via all'implementazione. Trigger tipici: "fammi una stima per CPQ-2298",
  "quanto costa questo ticket", "preparami il SOW per X", "prima di partire quanto ci vuole",
  "preventivo per il ticket X".
---

# sf-new-ticket-estimate
Workflow: analisi tecnica verificata + stima di effort + bozza SOW, senza creare branch né toccare codice.

## Quando usare questa skill
Quando l'utente vuole una stima/preventivo/SOW per un ticket Jira, prima di decidere se e quando avviare lo sviluppo — quindi prima di `sf-new-ticket`, non al suo posto.

## Comportamento

### 1. Leggi il ticket
Leggi `.claude/jira/<TICKET>.md`. Se non esiste, chiedi all'utente la descrizione o dove trovarla.

### 2. Analisi tecnica completa
Stessa profondità di `sf-new-ticket` step 2 — **non un pass superficiale sul solo testo del ticket**. Verifica il codice reale (leggi i file, non fidarti di nomi plausibili), conferma che i touchpoint ipotizzati esistano davvero, e cerca punti di impatto che il testo del ticket potrebbe non menzionare esplicitamente (le AS-IS descritte a parole spesso nascondono touchpoint tecnici distinti — verificalo nel codice, non dedurlo).

**Se la ricerca coinvolge più di 3 file o pattern non noti**: lancia un agente `Explore` con un prompt dettagliato (classi, trigger, metadata, pattern esistenti da estendere).
**Se la ricerca è semplice e localizzata**: usa direttamente Grep/Read senza agente.

Se emergono scelte architetturali con più opzioni valide, o comportamenti che il ticket non specifica, **fermati e discutine con l'utente** prima di proseguire — non assumere in silenzio.

### 3. Scrivi il piano tecnico
Stesso file e formato di `sf-new-ticket`: `.claude/plans/<TICKET>.md`. Se il ticket copre più MVP/fasi, usa sezioni separate nello stesso file (una per MVP), non file diversi.

### 4. Costruisci la tabella di stima
Lancia l'agente `sf-estimate-builder`, passandogli:
- il piano tecnico appena scritto
- la config di progetto `.claude/sf-estimate-config.md`, se esiste (template della tabella, regole di business come PM %, contingency)

Se la config non esiste, l'agente userà un default ragionevole — dopo la negoziazione con l'utente (step 5), proponi di salvarla in `.claude/sf-estimate-config.md` per i prossimi ticket.

### 5. Presenta e negozia
Mostra la tabella. **Il primo draft non è definitivo** — è normale che l'utente sposti voci, alzi/abbassi ore, separi in progetti indipendenti per MVP, tolga/aggiunga righe. Continua ad aggiustare a turni finché non è soddisfatto, prima di passare al punto 6.

### 6. Bozza SOW
Su richiesta, produci il contenuto per la sezione SOW (in blocco di codice, per copia facile): obiettivo, deliverable, esclusioni, assunzioni, rischi, criteri di accettazione, effort. Se ci sono parti non ancora decise dal business (open point espliciti), riportale come tali nel SOW, non come deliverable stimati.

Salva tabella + bozza SOW in `.claude/estimates/<TICKET>.md`.

### 7. Nessuna implementazione
Questa skill non crea branch e non tocca codice. Per procedere allo sviluppo, l'utente userà `sf-new-ticket`, che riuserà il piano tecnico già scritto qui invece di rifare l'analisi da zero (vedi step 0 di `sf-new-ticket`).
