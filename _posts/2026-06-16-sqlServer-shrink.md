---
title: "SQL Server: cosa succede realmente durante lo shrink di un data file"
date: 2026-06-19
categories: [SQL Server]
tags: [performance, tuning]
classes: article
layout: single
feedback: true
header:
  teaser: /assets/images/20260623sqlshrink/sqlshrinkk1.png
---

# SQL Server shrink

> **IMPORTANTE**
> Non usare lo shrink...

Lo shrink dei file ha l'obbiettivo di liberare lo spazio FISICO utilizzato dai file del database; più chiaramente si potrebbe pensare di effettuare un operazione di questo tipo nel momento in cui mi trovo in una situazione in cui vedo che il mio file: .mdf, .ldf o .ndf occupa una determinata quantità di spazio ma in relatà ne sta effettivamente utilizzando molto di meno.

<img width="811" height="217" alt="image" src="/assets/images/20260623sqlshrink/sqlshrinkk1.png" />

Vediamo un esempio sopra, dove vediamo in particolare che il Transaction Log ha tutto il suo spazio libero a disposizione. Attenzione, che però questa situazione è comune per questa tipologia di file.

Nonostante queste premesse possano far apparire lo shrink come un operazione estremamente utile, in realtà non è così; perchè come sappiamo i nostri database subiscono quotidianamente inserimenti, modifiche e cancellazioni e l'operazione di shrink può
- portare a una frammentazione degli indici: con peggioramento delle performance perchè sposta le pegine dati per compattare il file
- Crescita successiva del file: questo perchè ci possono essere casi in cui la crescita del file è dovuta a operazioni o attività del database; quindi creare una situazione in cui si effettua uno shrink (impattante) + operazione + nuovamente crescita del file, va a incidere sulle prestazioni
- Costi di CPU e IO: questo perchè sposta le pagine e aggiorna le allocazioni
- Blocchi e attese: si possono creare dei lock con conseguente rallentamento

> [!IMPORTANT]
> non utilizzare mai lo shrink come un operazione di manutenzione e non abilitare nemmeno auto-shrink su un database. Bisogna effettuarlo solo in caso in cui sono perfettamente a conoscenza del motivo dello spazio inutilizzato.

Questa operazione è da fare solo in casi particolari:
- ho determinato qual'è la causa della crescita dei miei file
-   per il transaction log può essere legato a transazioni da ottimizzare oppure a un database in modalità FULL su cui non viene effettuato il backup dei transaction log
-   per il file dati può essere necessario in caso di operazioni massive di rimozione dei dati
- casi di emergenza in cui devo recuperare dello spazio a livello di file system.

Nel caso in cui vado ad effettuarlo sui file dati mi devo sempre ricordare di effettuare la manutenzione degli indici.

Considera che in linea generale un transaction log dovrebbe essere il 25% della dimensione dei file di dati; attenzione però che questa non è una condizione sempre vera perchè dipende anche dall'attività sul database.


### Shrink Transaction Log

Ora come esempio vediamo il caso più comune di shrink, quello legato al transaction log.

In questi casi può essere necessario intervenire nel momento in cui la mia dimensione del file .ldf ha raggiunto una soglia >= del 50% del mio file dati.

Alcuni dei check che dovrò effettuare prima di intervenire:
- controllare il recovery model del mio database. Nel caso in cui il mio database sia in modalità FULL devo controllare che venga effettuato il backup dei log;
- controllare eventuali transazioni ancora attive, in questi casi il transaction log rimarrà aperto fino a quando le transazioni non sono committare;
- controllare se eventualmente ci sono attività che potrebbero portare a far ricrescere il mio transactio log dopo lo shrink;

Un'altro punto che consigliamo in questa attività è di controllare authogrowth dei database, in modo che in caso di crescite del file questa non sia:
- troppo bassa, quindi in caso di crescita l'operazione andrà effettuata più volte;
- troppo alta, che comporta dello spazio allocato ma non utilizzato.
  
<img width="1079" height="112" alt="image" src="/assets/images/20260623sqlshrink/sqlshrink2.png" />

Si consiglia solitamente di partire:
- file dati sui 256MB;
- file log sui 128MB.

Ora controllerò lo stato dei miei VLFs

```sql
-- per vedere lo stato dei VLF all'interno di uno specifico database

USE NomeDatabase;
GO
SELECT 
  file_id,
  vlf_sequence_number,
  vlf_begin_offset,
  vlf_size_mb,
  vlf_active,
  vlf_status
FROM sys.dm_db_log_info(DB_ID())
ORDER BY vlf_sequence_number;

-- OPPURE

USE NomeDatabase;
DBCC LOGINFO;
```

vlf_status può avere valori come 0 = inactive, 1 = initialized but unused, 2 = active.
Solo i VLF inattivi possono essere liberati da uno shrink.

<img width="484" height="195" alt="image" src="/assets/images/20260623sqlshrink/sqlshrink3.png" />

Una volta effettuato io controllo e visto che mie VFLs sono a 0 posso procedere.

ATTENZIONE :exclamation: effettuare sempre lo shrink dei file specifici non dell'interno database


<img width="733" height="346" alt="image" src="/assets/images/20260623sqlshrink/sqlshirnk4.png" />

Oppure con il comando TSQL
```sql
USE nameDB
GO
DBCC SHRINKFILE (N'nameDB_Log' , 20040)
GO
```

