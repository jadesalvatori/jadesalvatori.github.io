---
title: "PostgreSQL: quanto ci dobbiamo preoccupare delle dead tuple"
date: 2026-06-19
categories: [PostgreSQL]
tags: [performance, tuning, OOM]
classes: article
layout: single
feedback: true
---
# Out Of Memory

_Il problema di "Out of Memory" in PostgreSQL si verifica quando il database richiede più memoria di quella disponibile, causando il fallimento di operazioni specifiche o l'interruzione del processo che gestisce la query. È un segnale che le risorse hardware o i parametri di configurazione non sono adeguati al carico di lavoro._

PostgreSQL può consumare memoria in due modi principali:
- **memoria condivisa**: usata per le operazioni globali come la cache dei dati (*shared_buffers*) e il registro delle transazioni
- **memoria locale**: usata dai singoli processi di backend per gestire operazioni di ordinamento, join, creazione di indici ecc... Il parametro collegato è *work_mem*

Questo errore quindi può verificarsi quando una singola query richiede più memoria di quella assegnata, oppure il sistema esaurisce la memoria disponibile per i processi postgreSQL.

### Cause principali:
1. **configurazione della memoria inadeguata**: con i parametri work_mem, maintenance_work_mem o shared_buffer troppo alti oppure la memoria complessiva disponibile è insufficente rispetto al numero massimo di connessioni attive (mac_connections)
2. **query troppo pesanti**: query che elaborano grandi quanttà di dati, usano tabelle temporanee o richiedono ordinamenti complessi oppure join mal ottimizzate o query con subquery nidificate
3. **limiti del sistema operativo**: limiti su *shmmax* e *shmall* cioè limiti nella memoria condivisa, oppure, overcommit di memoria e linux potrebbe rifiutare allocazioni di memoria per i processi
4. **Uso di file temporanei**: quando una query supera il work_mem, postgresql utilizza i file temporanei su disco, ma se lo spazio su disco è insufficente o il numero di file temporanei è troppo alto può causare un errore.

### Impatto dell'errore
Si riscontrano query fallite, rallentamenti causati da un uso eccessivo di file temporanei o di swap che degrada le prestazioni. Infine si avrà un'instabilità del sistema con l'esaurimento della memoria fisica e inzia a utilizzare lo swap interno, così che il server potrebbe diventare lento o instabile.

>VERIFICHE
Il messaggio di errore viene registrato all'interno del file di log di postgresql solitamente contenuto nella directory /var/log/postgres/
In questi casi si consiglia di verificare le query che utilizzando file temporanei e le query attive

  ```sql
  SELECT datname, temp_files, temp_bytes
  FROM pg_stat_database;
  
  -- invece per le query attive
  SELECT pid, usename, query, state, wait_event, memory_bytes
  FROM pg_stat_activity;
  ```

## Soluzioni
1. revisione dei prametri
    - ridurre il valore di work_mem *
    - ottimizzare la shared_buffer, che non deve mai superare il 25/30% della RAM del sistema operativo
    - limita il numero di connessioni: un numero eccessivo di connessioni consuma memoria. RIduci il valore di max_connections e usa un connection pooler come pgBouncer
2. Ottimizzazione delle query
3. Aumento della memoria disponibile
     - aumento dei limiti di memoria condivisa del sistema operativo
     - abilita l'overcommit di memoria su linux
4. gestisci i file temporanei
    - aumento dello spazio su disco
    - spostare i file temporanei su un disco dedicato con il parametro *temp_tablespaces*
  
> [!NOTE]
> Perchè abbassare il valore *work_mem*

Abbassare il valore di work_mem in PostgreSQL può essere necessario per evitare il sovraccarico di memoria e per gestire efficacemente un elevato numero di connessioni simultanee, specialmente su server con risorse hardware limitate o su sistemi con molteplici query pesanti.

La work_mem è la quantità di memoria assegnata a ogni singola operazione temporanea (come ORDER BY, JOIN, e HASH AGGREGATE) e il suo valore viene assegnato ai PID specifici.
Ogni query complessa può avere più operazioni, e ciascuna di queste utilizza una *porzione separata di work_mem*. Se work_mem è impostato troppo alto, il consumo di memoria può rapidamente superare la memoria fisica disponibile, portando a:
- Out of Memory (OOM): Se il server esaurisce la memoria fisica.
- Uso eccessivo dello swap: Che rallenta le prestazioni.
- Instabilità del sistema: Quando il kernel deve terminare processi per liberare memoria.

>Si calcola:

- RAM = 64GB
- shared_buffer = 25GB
- max_connections = 2500, le connessioni che bisogna prendere in considerazione solo quelle che vengono create, quindi se ho mac connection a 2500 so che contemporaneamente saranno presenti circa 2000
- memoria sistema operativo
- maintenance_work_mem: 2GB

```
work_mem = RAM - shared_buffer - SO - mamaintenance_work_mem / connection*op
          = (64 - 25 - 8 - 2) / 2000
          = 0.015 * 1024 = 15MB ovviamente questo è un valore di riferimento
```

Il motivo per cui può essere necessario abbassare questo valore è per evitare il sovraccarico della memoria. Quando molte connessioni attive eseguono query simultaneamente, il consumo totale di memoria cresce linearmente. Se work_mem è impostato troppo alto, il consumo complessivo può superare la RAM disponibile. Infatti se facciamo 0.015 * 2000 = 30GB risultati dalla sottrazione.  
*Si consiglia sempre di tenere questo valore sempre tra i 4MB e i 16MB.*

>Quando abbassare work_mem

Abbassare work_mem è utile in questi casi:
- Molti utenti simultanei: Su server con alto numero di connessioni.
- RAM limitata: Su macchine con meno di 8 GB di RAM.
- Out of Memory frequente: Se il server raggiunge spesso i limiti di memoria.
- Query semplici: Quando la maggior parte delle query non beneficia di un valore elevato di work_mem.
  
Un valore più basso di work_mem comporta:
- Più utilizzo di file temporanei su disco: Quando PostgreSQL non ha abbastanza memoria, passa a memorizzare temporaneamente i dati sul disco. Questo può rallentare alcune query.
- Minor rischio di OOM: Le query consumano meno memoria fisica, riducendo i rischi di crash del sistema.
- Se hai necessità specifiche di migliorare le prestazioni per alcune query, puoi aumentare work_mem a livello di sessione o per singole query.

