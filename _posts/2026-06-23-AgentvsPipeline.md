---
title: "Intelligenza artificiale: Pipeline vs Agent"
date: 2026-06-19
categories: [data, ai]
tags: [teoria, agent, pipeline]
classes: article
layout: single
feedback: true
header:
  teaser: intelligenza-artificiale-spiegabile-xai.png
---

# Pipeline vs Agent AI

👉 **Pipeline (workflow) AI**
 = *“Io decido i passi, l’AI esegue pezzi intelligenti”*
 ✔ più semplice, controllabile, economica
 ✔ è la scelta giusta **nel 70–80% dei casi reali**

👉 **Agenti AI**
 = *“Do un obiettivo all’AI e lei decide cosa fare”*
 ✔ molto flessibili
 ❌ più complessi, meno prevedibili, più costosi
 ✔ utili solo in casi **aperti e non completamente definibili**

> Non è una moda: nel 2025–2026 **le pipeline dominano la produzione enterprise**, mentre gli agenti sono usati in casi più ristretti

------

## Cos’è una pipeline AI (workflow)

Una **pipeline** è una sequenza di passi **decisa da te**, **dove l’AI viene usata solo dove serve.**

Esempio mentale:

```
Input → Step 1 → Step 2 → Step 3 → Output
```

Con l’AI utilizzata in uno o più step.

### Esempio concreto

“Analizzo una richiesta cliente”

1. ricevo il testo
2. **LLM classifica** il tipo di richiesta
3. se è “incident”: apro ticket
4. se è “informazione”: genero risposta
5. salvo il risultato su DB

✅ L’AI **non decide il flusso**, decide solo *cosa* rispondere o classificare.

### Caratteristiche

- flusso **deterministico**
- stessi input → stessi step
- facile da testare e debuggare
- costi prevedibili (so quante chiamate LLM faccio)

Questo approccio è quello più comune in produzione 

### Quando usare una pipeline (quasi sempre all’inizio)

Usa una **pipeline AI** quando:

✅ i passi sono noti
✅ il processo è ripetibile
✅ vuoi controllo e tracciabilità
✅ devi andare in produzione
✅ sei alle prime armi

### Tipici casi da pipeline

- RAG (Q&A su documenti)
- classificazione / estrazione dati
- automazione IT / DB / ticketing
- reportistica
- ingestion + validazione + output

> Moltissimi casi etichettati come “agenti” **sono in realtà workflow mascherati** 

------

## Cos’è un agente AI

Un **agente AI** funziona così:

> “Ecco un obiettivo. Hai questi strumenti. Decidi tu cosa fare.”

Internamente gira in **loop**:

1. ragiona
2. decide un’azione
3. usa uno strumento (API, DB, search, codice…)
4. osserva il risultato
5. decide di nuovo se continuare o fermarsi

### Esempio concreto

“Sistema questo problema di pagamento di un cliente”

L’agente potrebbe:

- leggere i log
- interrogare il DB
- cercare policy interne
- provare più approcci
- fermarsi quando “ritiene” di aver risolto

👉 **Qui il percorso non è noto in anticipo**.


### Quando ha davvero senso un agente

Un **agente** ha senso SOLO se:

✅ **non puoi** predefinire tutti i passi
✅ il problema è esplorativo
✅ servono strategie diverse caso per caso
✅ il costo extra è giustificato

### Esempi realistici

- debugging automatico
- analisi investigativa
- ricerca complessa multi-step
- assistente tecnico che *decide* cosa interrogare

Anche qui: **pochi casi, ma giusti**

------

## Differenza di effort (sviluppo)

| Aspetto                | Pipeline | Agente       |
| ---------------------- | -------- | ------------ |
| Curva di apprendimento | ✅ bassa  | ❌ alta       |
| Design                 | lineare  | complesso    |
| Debug                  | facile   | difficile    |
| Governance             | semplice | obbligatoria |
| Manutenzione           | bassa    | alta         |

➡️ Per una **nofita**, partire da una pipeline è fortemente consigliato.

------

## Differenza di costo (importantissimo)

Dalle esperienze reali in produzione:

|              | Pipeline | Agente        |
| ------------ | -------- | ------------- |
| Chiamate LLM | 1–3      | 5–50+         |
| Costi token  | bassi    | imprevedibili |
| Latenza      | secondi  | minuti        |
| Rischio loop | no       | sì            |

Un errore comune è **pagare 10–50× di più** per un agente quando una pipeline basta 

------

## Regola d’oro (che quasi tutti imparano “dopo”)

> **Se puoi modellare il flusso → pipeline**
> **Se non puoi modellare il flusso → valuta un agente**

Molti team iniziano con agenti e **tornano indietro** verso pipeline più robuste

------

## Cosa ti consiglierei, da nofita

1. ✅ parti da **pipeline AI**
2. ✅ usa l’LLM come:
   - classificatore
   - estrattore
   - generatore
3. ❌ evita agenti finché:
   - non hai problemi di controllo
   - non senti il limite reale del workflow

------

# Esempio
## Mettiamo a confronto due casi uno di pipeline e uno di Agent

### CASO A — Pipeline AI

**“Gestione ticket di supporto database”**

📌 Problema

>Arrivano richieste testuali (mail / ticket / portale clienti) su problemi database.

Voglio:
- classificarle
- fare azioni coerenti
- avere controllo, costi prevedibili e auditabilità

🧠 **Come lo pensa un’architetta “pipeline”**

*“So già quali sono i passi del processo. L’AI mi aiuta, ma non guida.”*

🔗 Pipeline step-by-step
```
[Input ticket]
   ↓
[Normalizzazione testo]
   ↓
[LLM: classificazione]
   ↓
[Decisione a codice]
   ↓
[Azione tecnica]
   ↓
[Output + log]
```

✅ Cosa funziona molto bene
	✅ Sai sempre cosa succede
	✅ 1–2 chiamate LLM
	✅ Costo prevedibile
	✅ Facile da spiegare al cliente
	✅ Facile da certificare / auditare
	✅ Facile da mantenere

❌ Limiti (accettabili)
- Se il caso è molto ambiguo, serve intervento umano
- Devi conoscere il processo in anticipo

👉 Questo NON è un problema nel 90% dei use case enterprise.

### CASO B — Agente AI

**“Assistente tecnico autonomo per problemi database”**
- Stesso dominio.
- Stesso input.
**Approccio completamente diverso.**

**🧠 Come lo pensa un’architetta “agent”**
>“Non so in anticipo quali passi serviranno. Do un obiettivo all’AI e la lascio ragionare.”

**🧩 Obiettivo dell’agente**

“Analizza questa richiesta e risolvila nel modo migliore possibile”


🔁 Come lavora davvero l’agente (loop interno)
```
legge richiesta
↓
decide cosa fare
↓
usa uno strumento
↓
analizza il risultato
↓
decide il prossimo passo
↓
(ripete)
```

**🧠 Cosa potrebbe fare l’agente (senza che tu lo sappia prima)**
- classificare il problema
- cercare nella knowledge base
- cambiare ipotesi
- controllare metriche
- fare altre query
- rivalutare
- decidere se aprire ticket o no
- fermarsi *quando lo ritiene opportuno*

**✅ Quando brilla davvero**
- ✅ problemi **esplorativi**
- ✅ troubleshooting complesso
- ✅ investigazioni tecniche
- ✅ debugging non strutturato

**❌ Costi e rischi REALI**

- ❌ **5–30 chiamate LLM**
- ❌ comportamento variabile
- ❌ difficile da debuggare
- ❌ difficile da spiegare a un audit o cliente
- ❌ difficile da testare


# 🔴 CONFRONTO DIRETTO (stesso problema)

| Aspetto               | Pipeline       | Agent      |
| --------------------- | -------------- | ---------- |
| Controllo             | ✅ Totale       | ❌ Limitato |
| Flusso                | Deterministico | Dinamico   |
| Chiamate LLM          | 1–2            | 5–30+      |
| Costi                 | Prevedibili    | Variabili  |
| Debug                 | Facile         | Complesso  |
| Produzione enterprise | ✅ Standard     | ⚠️ Raro     |
| Adatto a nofita       | ✅ Sì           | ❌ No       |


# 🧠 Regola pratica che voglio ti resti impressa

> **Se puoi spiegare il processo in un diagramma → pipeline**
> **Se NON riesci a disegnarlo → forse serve un agente**

E soprattutto:

> **Molti “agenti” in vendita oggi sono pipeline travestite.**


# Diagramma visivo: Pipeline AI vs Agent AI

## 🔹 Diagramma A — Pipeline AI (workflow deterministico)

```
┌──────────┐
│  Input   │  (ticket / testo / richiesta)
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Preprocessing│  (pulizia, validazioni)
└────┬─────────┘
     │
     ▼
┌──────────────┐
│     LLM      │  (classifica / riassume / estrae)
│ (task unico) │
└────┬─────────┘
     │
     ▼
┌──────────────┐
│ Business     │  (if/else, routing, soglie)
│ Logic (code) │
└────┬─────────┘
     │
     ▼
┌──────────────┐
│   Output     │  (ticket, risposta, DB)
└──────────────┘
```

### Cose chiave da notare

- ✅ **il flusso è deciso dal codice**
- ✅ l’LLM è **uno step**, non il regista
- ✅ stesso input → **stesso percorso**
- ✅ facilmente testabile e monitorabile

Questo tipo di architettura è definito come *LLM workflow / pipeline deterministica* ed è quella che domina i sistemi AI in produzione perché è prevedibile e governabile.

## 🔸 Diagramma B — Agent AI (controllo nel modello)

```
┌──────────┐
│  Input   │
└────┬─────┘
     │
     ▼
┌─────────────────────────┐
│        AGENT            │
│  ┌──────────────────┐  │
│  │ Reason / Decide  │◄─┐│
│  └──────┬───────────┘  ││
│         │              ││
│         ▼              ││
│  ┌──────────────┐     ││
│  │ Use Tool     │─────┘│
│  │ (API, DB…)   │      │
│  └──────┬───────┘      │
│         │              │
│         ▼              │
│   Observe Result ──────┘
│       (loop)
└─────────┬──────────────┘
          │
          ▼
      ┌────────┐
      │ Output │
      └────────┘
```

### Cose chiave da notare

- ❌ **il flusso non è noto in anticipo**
- ❌ stesso input → percorsi diversi
- ✅ molto flessibile
- ❌ costi, latenza e debug meno prevedibili

Questa struttura è quella tipica degli *LLM agents*, dove il modello controlla il ciclo decisionale e l’uso degli strumenti.

------

## 🔍 Confronto visivo “a colpo d’occhio”

| Aspetto               | Pipeline    | Agent       |
| --------------------- | ----------- | ----------- |
| Chi decide il flusso  | Codice      | LLM         |
| Numero step           | Fisso       | Variabile   |
| Loop                  | No          | Sì          |
| Debug                 | Lineare     | Complesso   |
| Costi                 | Prevedibili | Variabili   |
| Produzione enterprise | ✅ standard  | ⚠️ selettivo |

------

## Quando una pipeline **non basta più davvero**

Questa è la parte **più importante**, perché evita errori costosi.

❌ *Non basta più* **non** significa:
- “ci hanno parlato di agenti”
- “è più figo”
- “LangChain lo supporta”

✅ Significa **problemi strutturali reali**.

### 🔴 Segnale 1 — Non riesci a disegnare il flusso completo

Se inizi a dire:

- “dipende”
- “vediamo cosa esce”
- “forse prima A, forse B”

➡️ il flusso **non è più completamente prevedibile**
➡️ pipeline rigida inizia a forzare il problema

### 🔴 Segnale 2 — Servono iterazioni per raggiungere un risultato

Esempi:
- troubleshooting complesso
- investigazioni tecniche
- ricerca multi‑step

Qui il sistema deve:
- formulare ipotesi
- verificarle
- cambiare strategia

Questo tipo di *reason → act → observe → repeat* è tipico degli agenti

### 🔴 Segnale 3 — Le domande successive dipendono dalle risposte precedenti

Esempio classico:

> “Perché il database è lento?”

- prima guardi metriche
- poi decidi se servono altre query
- poi fai follow‑up
- poi rivaluti

Non stai più solo “eseguendo step”, stai **guidando un’indagine**.

### 🔴 Segnale 4 — Devi usare molti strumenti in modo adattivo

- API
- query DB
- documentazione
- metriche
- log

E **non sai prima quali** serviranno.

Questo è uno dei pochi casi in cui l’agente ha senso reale.

## 🧠 Pattern IMPORTANTISSIMO (quasi nessuno lo dice)

> **Molti sistemi production-grade NON passano direttamente da pipeline → agent**
>  **passano prima da pipeline → pipeline avanzata**

## ✅ Cosa provare **prima** degli agenti

Prima degli agenti, si usano:

- routing (LLM decide *quale pipeline*)
- orchestrator + worker
- evaluator / validator
- human‑in‑the‑loop
- retry controllati

Tutto questo resta **pipeline-driven**, ma più potente

## 🧭 Regola finale da portarti via

> **Pipeline**: processi noti, ripetibili, auditabili
> **Agent**: esplorazione, indagine, autonomia controllata

E soprattutto:

> **Se puoi ancora mettere `if / else`, probabilmente NON ti serve un agente.**


# Casi borderline
Ci possono essere casi che risultano borderline, una caso di esempio può essere un rallentamento in un database PostgreSQL dove l'investigazione per la risoluzione può essere:

- può essere semplice
- può diventare investigativo

**Caso con pipeline **

> “Gestisco l’80% dei casi in automatico, il resto lo mando a umano o escalation.”

*✅ Pro*
	✅ deterministica
	✅ auditabile
	✅ costi bassi
	✅ perfetta per MSP e produzione
*❌ Contro*
	❌ non esplora cause “creative”
	❌ non formula ipotesi

**Caso con agent **

> “Non so prima cosa guardare. Lo capisco man mano.”

*✅ Pro*
	✅ investigazione profonda
	✅ adatto a troubleshooting complesso
	✅ flessibile
*❌ Contro*
	❌ costi molto variabili
	❌ difficile da spiegare al cliente
	❌ difficile da certificare

#### 🔥 Confronto finale sul caso borderline

| Aspetto        | Pipeline avanzata | Agent     |
| -------------- | ----------------- | --------- |
| Copertura casi | 70–85%            | 90–95%    |
| Costi          | bassi             | alti      |
| Debug          | semplice          | complesso |
| Adatta a MSP   | ✅ sì              | limitata  |
| Autonomia      | bassa             | alta      |

#### 🧠 Strategia “da architetta esperta” (IMPORTANTISSIMA)

> **Pipeline FIRST, agent LATER (e solo per sotto‑compiti)**

Pattern reale usato in produzione:

- pipeline principale
- agent solo per i casi che “non tornano”

Questo è il compromesso più maturo visto oggi nei sistemi enterprise

#### ✅ Messaggio finale da portarti via

> ⚠️**Gli agenti non sostituiscono le pipeline.**
> 🧠**Le pipeline sono l’infrastruttura.**
> ⚠️**Gli agenti (forse) un’estensione.**


































AspettoPipelineAgentControllo✅ Totale❌ LimitatoFlussoDeterministicoDinamicoChiamate LLM1–25–30+CostiPrevedibiliVariabiliDebugFacileComplessoProduzione enterprise✅ Standard⚠️ RaroAdatto a nofita✅ Sì❌ No

🧠 Regola pratica che voglio ti resti impressa

Se puoi spiegare il processo in un diagramma → pipeline
Se NON riesci a disegnarlo → forse serve un agente

E soprattutto:

Molti “agenti” in vendita oggi sono pipeline travestite.
