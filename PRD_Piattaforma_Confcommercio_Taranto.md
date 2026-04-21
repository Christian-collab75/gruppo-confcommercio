# PRD — Piattaforma Gestione Corsi Confcommercio Taranto

**Versione:** 0.1 draft
**Autore:** Securitix Solutions
**Destinatario finale del prodotto:** Confcommercio Taranto
**Ultimo aggiornamento:** 16 aprile 2026

---

## 1. Obiettivo

Realizzare una **piattaforma web unificata** per la gestione amministrativa dei corsi di formazione in presenza erogati da Confcommercio Taranto con fondi della Regione Puglia.

La piattaforma non è un LMS. È un **tool gestionale** che digitalizza tre processi oggi manuali o cartacei:

1. Registro presenze giornaliero
2. Comunicazioni e calendario
3. Distribuzione dei materiali didattici

**Obiettivo strategico Securitix:** vincere Confcommercio Taranto come primo cliente di riferimento per replicare la soluzione presso altre sedi Confcommercio e altri enti di formazione finanziata in Puglia.

---

## 2. Scope

### In scope
- Gestione anagrafica corsi, classi, corsisti, docenti
- Registro presenze digitale conforme alla rendicontazione Regione Puglia
- Comunicazioni automatiche via email (convocazioni, reminder, variazioni)
- Calendario condiviso e aggiornato in tempo reale
- Area riservata corsisti per accedere ai materiali della propria classe
- Dashboard amministrativa per lo staff Confcommercio
- Esportazione dati per rendicontazione

### Out of scope (v1)
- Erogazione di contenuti e-learning (video on-demand, SCORM, quiz valutativi)
- Tracking del tempo di studio o progressione didattica
- Integrazione profonda con sistemi di videoconferenza (basta un link nel calendario)
- App mobile nativa (la webapp deve essere responsive, ma niente iOS/Android store in v1)
- Pagamenti, fatturazione, gestione contabile
- Integrazione con SIRFO / sistemi regionali di rendicontazione automatica (valutazione per v2)

---

## 3. Vincoli e requisiti trasversali

### Vincoli normativi (bloccanti)
- **GDPR:** trattamento dati personali dei corsisti. Da produrre: informativa privacy, registro trattamenti, valutazione necessità DPIA.
- **Rendicontazione Regione Puglia (FSE+):** il registro presenze digitale deve essere conforme ai requisiti degli avvisi pubblici di riferimento. **⚠ Da verificare puntualmente prima del kick-off:** modalità di firma ammesse (FEA, firma grafometrica, firma semplice con log di autenticazione, ecc.). Questa verifica condiziona tutte le scelte architetturali del Workstream A.
- **Accessibilità:** non obbligatoria per soggetto privato, ma WCAG 2.1 AA raccomandato come standard di qualità.

### Vincoli operativi
- Volume: ~1 corso/mese, max 20 corsisti per corso (~240 utenti unici/anno previsti)
- Competenze tecniche dello staff Confcommercio: basse → UX molto semplice, configurazione autonoma dei corsi
- Budget: soluzione economicamente sostenibile alla scala attuale ma architettata per scalare 10–50x senza refactoring

### Principi di prodotto
- **Assemblare, non costruire da zero:** usare servizi gestiti (auth, email, storage, firma digitale) invece di sviluppare componenti proprietari quando non danno valore differenziante
- **Semplicità prima di completezza:** se una feature non risolve uno dei tre problemi prioritari, va tagliata
- **Zero carta:** ogni output deve essere esportabile digitalmente in formato accettato dalla Regione

---

## 4. Personas

| Persona | Esigenza principale | Frequenza d'uso |
|---|---|---|
| **Admin Confcommercio** | Creare corsi, gestire iscrizioni, monitorare presenze, esportare dati per rendicontazione | Quotidiana |
| **Docente** | Vedere la propria agenda, aprire il registro della giornata, caricare i materiali della lezione | Nei giorni di lezione |
| **Corsista** | Ricevere comunicazioni, consultare il calendario, firmare presenza, scaricare materiali | Nei giorni di lezione |

---

## 5. Architettura proposta (da validare al kick-off)

**Approccio raccomandato:** light custom stack basato su servizi gestiti.

- **Frontend:** Next.js (React) — unica codebase per web app responsive
- **Backend + DB + Auth + Storage:** Supabase (PostgreSQL managed, auth integrata, storage per materiali)
- **Email transazionali:** Resend o SendGrid
- **Firma digitale presenze:** da decidere in base alla verifica normativa (opzioni: Yousign, Namirial, soluzione OTP custom, QR code + log)
- **Hosting:** Vercel (frontend) + Supabase cloud (backend)
- **Dominio/sottodominio:** da assegnare

**Perché questo stack:** time-to-market rapido, costi fissi bassi (<100€/mese a questo volume), nessuna infrastruttura da gestire, scalabilità orizzontale garantita dai provider, skill diffuse sul mercato.

**Alternativa no-code considerata e scartata:** Airtable + Softr + Make avrebbe tempi di prototipazione più rapidi ma limiti duri sulla personalizzazione della firma presenze e sulla brandizzazione per la rivendita a terzi.

---

## 6. WORKSTREAM A — Registro Presenze Digitale

**Team:** Gruppo A (lead: da nominare)
**Priorità:** 🔴 Massima — è il differenziatore del prodotto
**Stima effort indicativa:** 40–60% del totale

### Obiettivo
Sostituire completamente il registro cartaceo con un sistema digitale **conforme ai requisiti di rendicontazione della Regione Puglia**, veloce da compilare in aula e che produca un output esportabile valido ai fini del controllo.

### Flusso utente target
1. Il docente apre l'app in aula su tablet o PC
2. Vede la lista dei corsisti iscritti alla lezione del giorno
3. Ogni corsista, uno alla volta (o in modalità batch), conferma la propria presenza tramite il meccanismo di firma adottato
4. Il registro si chiude a fine lezione generando un PDF firmato digitalmente con: corso, data, orario inizio/fine, elenco presenti/assenti, firma del docente
5. Il PDF è archiviato e disponibile per l'export massivo in fase di rendicontazione

### Requisiti funzionali
- CRUD lezioni (associate a un corso e a una data)
- Apertura/chiusura registro con timestamp
- Firma presenza corsista (meccanismo da definire — vedi sotto)
- Firma docente a chiusura
- Generazione PDF del registro giornaliero
- Storico registri per corso / per corsista
- Export massivo per rendicontazione (PDF + CSV)
- Gestione assenze giustificate con note

### Decisioni aperte (bloccanti)
**Modalità di firma del corsista.** Quattro opzioni candidate, da valutare dopo la verifica normativa:

| Opzione | Pro | Contro |
|---|---|---|
| **Firma grafometrica su tablet** | Percezione "naturale", simile al cartaceo | Richiede hardware dedicato, dubbi sul valore legale senza certificazione FEA |
| **OTP via SMS/email + log IP** | Zero hardware, veloce | Da verificare se Regione Puglia lo accetta per corsi FSE+ |
| **QR code personale + PIN** | Veloce, scalabile | Idem sopra |
| **FEA tramite provider certificato** (Namirial, Yousign) | Massima certezza giuridica | Costo per firma, setup più complesso |

👉 **Azione prerequisito:** verifica documentale sugli avvisi FSE+ Regione Puglia vigenti e consultazione informale con ufficio rendicontazione Confcommercio.

### Deliverable
- Modulo registro presenze integrato nella piattaforma
- Template PDF registro giornaliero conforme
- Procedura di export per rendicontazione
- Documentazione legale della modalità di firma scelta

### Definition of Done
- Un corso pilota interno è stato gestito integralmente senza carta
- Il PDF generato è stato validato informalmente dall'ufficio rendicontazione Confcommercio
- Export funzionante per almeno 10 lezioni simulate

### Interfacce con altri workstream
- **Riceve da Workstream B:** calendario lezioni (una lezione nel registro = un evento nel calendario)
- **Riceve da Fondamenta:** anagrafica corsisti e docenti

---

## 7. WORKSTREAM B — Comunicazioni & Calendario

**Team:** Gruppo B (lead: da nominare)
**Priorità:** 🟡 Alta
**Stima effort indicativa:** 20–30% del totale

### Obiettivo
Eliminare la comunicazione manuale (telefonate, email scritte a mano) sostituendola con notifiche automatiche innescate dagli eventi del corso, e fornire un calendario unico sempre aggiornato.

### Flusso utente target
- L'admin crea un corso con il suo calendario lezioni
- Al momento dell'iscrizione ogni corsista riceve una convocazione automatica con il calendario
- 24h prima di ogni lezione parte un reminder automatico
- Se una lezione viene modificata (orario, aula, link), tutti i corsisti e il docente ricevono una notifica automatica di variazione
- Il calendario è consultabile sempre e ovunque, e sincronizzabile con il calendario personale del corsista (file .ics)

### Requisiti funzionali
- Editor calendario lezioni per admin (drag & drop o form semplice)
- Invio email transazionali con template per: convocazione, reminder 24h, variazione, annullamento
- Template email brandizzati Confcommercio, editabili dall'admin senza intervento tecnico
- Vista calendario per corsista (mensile/settimanale)
- Export .ics per integrazione con Google Calendar / Outlook / Apple Calendar
- Log di tutte le comunicazioni inviate (audit trail)

### Requisiti non funzionali
- Deliverability email > 98% (usare provider reputato, SPF/DKIM/DMARC configurati)
- Tempo di invio notifica variazione < 2 minuti dall'evento

### Decisioni aperte
- **SMS o WhatsApp come canale aggiuntivo per reminder?** Costo SMS ~0,05€/msg × 20 corsisti × 1 reminder/lezione × ~20 lezioni/corso = ~20€/corso. Sostenibile. Da valutare in v1 o v2.
- **Sincronizzazione bidirezionale con Google Calendar?** Complica molto lo sviluppo. Propendo per export .ics mono-direzionale in v1.

### Deliverable
- Modulo calendario con editor admin
- Sistema di template email
- Automazioni per i 4 eventi chiave (convocazione, reminder, variazione, annullamento)
- Export .ics funzionante

### Definition of Done
- Un corso pilota è stato comunicato end-to-end senza email scritte manualmente
- 100% delle notifiche di test sono state consegnate entro SLA
- Un corsista non tecnico è riuscito a importare il calendario sul proprio telefono senza assistenza

### Interfacce con altri workstream
- **Fornisce a Workstream A:** lista lezioni del giorno con corsisti attesi
- **Fornisce a Workstream C:** lista lezioni cui associare i materiali
- **Riceve da Fondamenta:** anagrafica e contatti

---

## 8. WORKSTREAM C — Area Corsisti & Materiali Didattici

**Team:** Gruppo C (lead: da nominare)
**Priorità:** 🟢 Media
**Stima effort indicativa:** 15–25% del totale

### Obiettivo
Fornire un'area riservata dove ogni corsista può accedere ai materiali delle lezioni del proprio corso, e dove il docente può caricarli in modo semplice.

### Flusso utente target
- Il docente, a fine lezione (o prima), carica slide/dispense associandole alla lezione del giorno
- Il corsista accede all'area riservata, vede la lista delle lezioni del proprio corso, e per ogni lezione può scaricare i materiali
- Il materiale resta accessibile per tutta la durata del corso + N mesi configurabili

### Requisiti funzionali
- Upload file da parte del docente (PDF, PPT, DOCX, immagini, ZIP — max 50 MB/file)
- Associazione materiale → lezione → corso
- Area riservata corsista con login
- Download autenticato (no link pubblici)
- Log accessi ai materiali (utile per rendicontazione e per capire l'engagement)
- Sezione "Lezioni svolte" con diario giornaliero: argomenti trattati + materiali + presenti

### Requisiti non funzionali
- Storage: stima ~500 MB/corso → ~6 GB/anno. Ampiamente gestibile.
- Permessi: un corsista vede SOLO i materiali del proprio corso

### Deliverable
- Area riservata corsista con autenticazione
- Modulo upload materiali lato docente
- Diario lezioni svolte
- Log accessi

### Definition of Done
- 100% dei materiali di un corso pilota caricati e scaricati correttamente
- Un corsista test non è riuscito ad accedere ai materiali di un corso a cui non è iscritto (test di isolamento)

### Interfacce con altri workstream
- **Riceve da Workstream B:** struttura corsi/lezioni cui appendere i materiali
- **Riceve da Workstream A:** elenco presenti per la sezione "Lezioni svolte"
- **Riceve da Fondamenta:** autenticazione corsista

---

## 9. Fondamenta condivise (cross-team)

Queste componenti non appartengono a nessun workstream ma devono essere pronte per primi, altrimenti i tre team si bloccano. Richiedono un **owner tecnico trasversale** (tech lead o il team A che ha lo scope più critico).

- **Setup progetto:** repository, CI/CD, ambienti (dev/staging/prod), dominio
- **Autenticazione:** login admin/docente/corsista, gestione ruoli, password reset
- **Modello dati core:** entità `Corso`, `Lezione`, `Corsista`, `Docente`, `Iscrizione`
- **Design system minimo:** palette Confcommercio, componenti UI di base, layout responsive
- **Impianto GDPR:** informativa privacy, cookie banner, gestione consensi, meccanismo di cancellazione dati

**Raccomandazione:** dedicare la prima settimana di lavoro alle Fondamenta **prima** di far partire i tre workstream in parallelo. Riduce drammaticamente i conflitti d'integrazione.

---

## 10. Milestone e integrazione

| Settimana | Attività |
|---|---|
| **W1** | Fondamenta condivise. Verifica normativa firma presenze. Kick-off team. |
| **W2–W4** | Sviluppo parallelo dei 3 workstream. Sync settimanale tra lead. |
| **W5** | Integrazione end-to-end. Bug fix. Test su corso pilota simulato. |
| **W6** | Demo a Confcommercio. Raccolta feedback. |
| **W7–W8** | Hardening, documentazione utente, training staff. Go-live su primo corso reale. |

Stima complessiva: **~8 settimane** per arrivare al primo corso live. La demo pitchabile a Confcommercio può essere pronta a W5.

---

## 11. KPI di successo

Misurabili sul primo corso live vs. l'ultimo corso gestito col vecchio metodo:

- **Tempo di compilazione registro giornaliero:** target −80%
- **Email inviate manualmente dallo staff:** target −95%
- **Richieste "dove trovo il materiale della lezione X?":** target −90%
- **Errori/incompletezze nel registro rilevati in rendicontazione:** target = 0
- **NPS corsisti sulla comunicazione del corso:** target > 50

---

## 12. Rischi principali

| Rischio | Impatto | Mitigazione |
|---|---|---|
| La modalità di firma scelta non è accettata in rendicontazione | 🔴 Critico | Verifica documentale + validazione informale con ufficio rendicontazione **prima** di iniziare lo sviluppo del Workstream A |
| Confcommercio chiede modifiche di scope durante lo sviluppo | 🟡 Alto | PRD firmato in kick-off, change request formali per modifiche |
| I tre team lavorano su modelli dati incompatibili | 🟡 Alto | Fondamenta condivise consolidate in W1, schema DB versionato e unico |
| Bassa adozione da parte dei corsisti meno digitalizzati | 🟡 Medio | UX estremamente semplice, guida cartacea di onboarding al primo incontro |
| Vendor lock-in su Supabase/Vercel | 🟢 Basso | Stack standard (PostgreSQL, Next.js), migrazione possibile se necessario |

---

## 13. Decisioni aperte in attesa di input

1. Modalità firma presenze (dipende da verifica normativa Regione Puglia)
2. Lead tecnico dei tre gruppi
3. Budget indicativo per servizi terzi (firma digitale, email, hosting)
4. Timing del pitch a Confcommercio: demo o prodotto già funzionante?
5. Modello commerciale con Confcommercio: licenza annuale? Canone mensile? Setup + maintenance?

---

*Questo PRD è una v0.1. Va validato dal team in kick-off e aggiornato iterativamente. Ogni modifica materiale richiede un bump di versione e una nota di change log.*
