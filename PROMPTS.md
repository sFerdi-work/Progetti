# Prompt library — NM Coaching Pro

Prompt pronti da incollare in Claude Code, adattati a **questo** repository
(PWA single-file con Supabase + Chart.js, convertita in APK con PWABuilder).

Sono costruiti sui pattern della [Prompt library ufficiale](https://code.claude.com/docs/en/prompt-library)
e delle [Best practices](https://code.claude.com/docs/en/best-practices.md).
Sotto ogni prompt, **Perché funziona** indica il pattern usato, così puoi scriverne di tuoi.

---

## I 6 pattern (in breve)

| Pattern | Cosa significa | Errore tipico che evita |
|---|---|---|
| **Risultato, non passi** | Dici cosa vuoi, Claude trova i file | Micro-gestire e sbagliare il file |
| **Auto-verifica** | Chiedi di eseguire/testare/confrontare | Claude si ferma al primo tentativo |
| **Riferimento** | "Come la pagina X" | Codice incoerente con l'esistente |
| **Target misurabile** | Metrica + soglia | "Fatto" ambiguo |
| **Dai l'artefatto** | Incolla errori, log, screenshot, `@file` | Claude lavora sulla tua parafrasi, non sulla fonte |
| **Formato della risposta** | Formato, lunghezza, pubblico | Risposta giusta ma inutilizzabile |

### Cosa manca alla guida ufficiale (aggiunte mie)

1. **Vincoli e non-obiettivi.** Dire cosa *non* toccare conta quanto dire cosa fare.
   In un file HTML da 260 KB, "non riformattare il resto del file" evita diff illeggibili.
2. **Criterio di stop.** "Se trovi più di una causa possibile, fermati e chiedimi" evita
   correzioni speculative.
3. **Un obiettivo per prompt.** Prompt multi-obiettivo producono commit misti difficili da revertare.
4. **Plan mode per le modifiche rischiose** (schema Supabase, service worker, cache):
   vedi l'elenco dei file prima di qualsiasi modifica.

---

## 1. Struttura e deploy

### Verifica integrità del repository

```text
controlla che ogni file del repo corrisponda al suo nome e a quanto descritto nel README
(tipo reale, dimensioni immagini, contenuto). Elenca le discrepanze in una tabella
"nome attuale → contenuto reale → nome corretto" e non rinominare nulla finché non confermo.
```

**Perché funziona:** *auto-verifica* (`file`, dimensioni reali) + *formato della risposta* (tabella)
+ *criterio di stop*. Caricare file dal sito GitHub può scambiare i nomi senza errori visibili.

### Audit PWA prima di PWABuilder

```text
servi il sito in locale, aprilo con Playwright e verifica che manifest e service worker
siano validi come li controllerebbe PWABuilder: icone 192/512 presenti e delle dimensioni
dichiarate, maskable corrette, sw.js registrato e attivo, start_url raggiungibile offline.
Dammi un elenco pass/fail.
```

**Perché funziona:** *target misurabile* (criteri PWABuilder) + *auto-verifica* in un browser reale.

---

## 2. Bug e debug

### Errore a runtime

```text
quando salvo una nuova misurazione compare questo errore in console:
<incolla stack trace>
trova la causa, correggila con la modifica minima e dimostra con Playwright che il salvataggio
ora funziona. Non riformattare il resto del file.
```

**Perché funziona:** *dai l'artefatto* (stack trace reale) + *auto-verifica* + *vincolo*.

### Bug di cache del service worker

```text
dopo un aggiornamento gli utenti vedono ancora la versione vecchia dell'app.
Analizza la strategia di cache in sw.js e spiegami perché succede, poi proponi la correzione
(senza applicarla) con i pro e i contro di ciascuna alternativa.
```

**Perché funziona:** separa *diagnosi* da *intervento*: il service worker sbagliato può bloccare
l'app sui telefoni degli utenti, quindi prima capire, poi decidere.

---

## 3. Nuove funzionalità

### Nuova sezione coerente con l'esistente

```text
aggiungi una sezione "Circonferenze nel tempo" nella pagina grafici, con lo stesso stile
e la stessa logica Chart.js dei grafici già presenti. Leggi i dati dalla stessa tabella
Supabase usata per le misure. Verifica in locale che il grafico si disegni con dati di esempio.
```

**Perché funziona:** *riferimento* (grafici esistenti) + *risultato, non passi* + *auto-verifica*.

### Modifica allo schema dati

```text
voglio salvare anche la frequenza cardiaca a riposo per ogni cliente.
Usa plan mode: mostrami prima quali tabelle Supabase, query e form andrebbero toccati,
e lo script SQL di migrazione. Nessuna modifica finché non approvo.
```

**Perché funziona:** modifica rischiosa e difficile da annullare → *plan mode* esplicito.

---

## 4. Qualità e prestazioni

### Peso e tempo di caricamento

```text
misura il tempo di primo caricamento su rete 3G simulata con Playwright.
Portalo sotto i 3 secondi e mostrami cosa hai cambiato e il prima/dopo.
```

**Perché funziona:** *target misurabile* con soglia e metodo di misura dichiarati.

### Sicurezza Supabase

```text
controlla come l'app usa Supabase: quale chiave è nel client, se le tabelle dipendono
dalle Row Level Security e se un utente potrebbe leggere i dati di altri clienti.
Rispondi con un elenco di rischi ordinati per gravità, ciascuno con la correzione suggerita.
```

**Perché funziona:** *formato della risposta* (lista ordinata per gravità). In un'app client-side
la chiave anon è pubblica per definizione: la protezione reale sono le policy RLS.

---

## 5. Capire il codice

```text
spiegami come funziona il calcolo della composizione corporea da pliche cutanee:
quale formula usa, da dove prende i dati e dove viene mostrato. Rispondi in 10 righe
massimo, per un personal trainer, non per un programmatore.
```

**Perché funziona:** *formato della risposta* con lunghezza e pubblico espliciti.

---

## Rendere i prompt ripetibili

- Quando un prompt funziona, salvalo come [skill](https://code.claude.com/docs/en/skills.md)
  (es. `/audit-pwa`) così si lancia con un comando.
- Le convenzioni del progetto (single-file HTML, nessun build step, stile CSS a variabili,
  deploy su GitHub Pages) vanno in un `CLAUDE.md`: ogni sessione parte con quel contesto.
