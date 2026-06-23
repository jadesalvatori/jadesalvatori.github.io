---
title: "PostgreSQL: quanto ci dobbiamo preoccupare delle dead tuple"
date: 2026-06-19
categories: [PostgreSQL]
tags: [performance, tuning]
classes: article
layout: single
feedback: true
---

# Dead Tuple

Per individuare il numero di dead tuples in una tabella in PostgreSQL, puoi utilizzare le statistiche del sistema fornite dalla vista pg_stat_user_tables. Ecco come fare:

```sql
	SELECT 	relname AS table_name,
	    	n_dead_tup AS dead_tuples,
	    	n_live_tup AS live_tuples,
	    	pg_size_pretty(pg_total_relation_size(relid)) AS total_size
	FROM pg_stat_user_tables
	WHERE pg_total_relation_size(relid) > 48240256 -- dsi 50MB
			--AND relname = 'nomeTabella' --se voglio filtrare per tabella
	ORDER BY n_dead_tup DESC;
```

Dove
- relname: Il nome della tabella.
- n_dead_tup: Numero di dead tuples nella tabella (tuple obsolete).
- n_live_tup: Numero di tuple attive (non obsolete) nella tabella.
- pg_total_relation_size: Dimensione totale della tabella (inclusi indici e TOAST).

La seguente query ti mostra lo stato delle tuple a livello di database.
Si può selezionare sia lo schema che eventualmente id delle table; fornendo anche un percentuale sulle dead tuple

```sql
		SELECT 
		    schemaname,
			relname AS table_name,
		    n_dead_tup AS dead_tuples,
		    n_live_tup AS live_tuples,
		    pg_size_pretty(pg_total_relation_size(relid)) AS total_size,
			ROUND((n_dead_tup * 100.0) / (n_live_tup + n_dead_tup), 2) ||'%' AS dead_tuples_percentage, --  Se desideri mostrare solo le tabelle con 										--una percentuale significativa di dead tuples (ad esempio, superiore al 5%)
			last_vacuum, 
			last_autovacuum,
			last_autoanalyze
		FROM 
		    pg_stat_user_tables
		WHERE   n_dead_tup != 0 
			and n_live_tup + n_dead_tup > 0 -- Per evitare divisione per zero
			and ROUND((n_dead_tup * 100.0) / (n_live_tup + n_dead_tup), 2) > 5 -- Filtra le tabelle con più del 5% di tuple morte
		ORDER BY 
		    n_dead_tup desc

```

## Tabelle TOAST 
Sono tabelle che troviamo all'interno di postgreSQL che contengono dati di gandi dimensioni come img, audio ... Vengono suddivise dalle tabelle con lo scopo di migliorare le performance.

    select relname from pg_class where oid = (select reltoastrelid from pg_class where relname=’TABLE_NAME’)


## Indici
La seguente query invece restituisce le statistiche degli indici presenti in un database selezionando le tabelle 

    select pg.schemaname as nome_schema,
    		pg.relname as nome_tabella,
    		pg.indexrelname as nome_indice,
    		pg_size_pretty(pg_relation_size(pg.relid)) as "Table Size",
     		pg_size_pretty(pg_relation_size(pg.indexrelid)) "Index Size",	
    		pg.idx_scan, -- Numero di scansioni dell'indice avviate su questo indice
    		pg.idx_tup_read, -- Numero di voci di indice restituite dalle scansioni su questo indice
    		pg.Idx_tup_fetch, -- Numero di righe di tabella attive recuperate da semplici scansioni dell'indice utilizzando questo indice
    		i.indexdef
    from pg_stat_user_indexes as pg
    inner join pg_indexes as i on i.schemaname = pg.schemaname
    			and i.tablename = pg.relname
    			and i.indexname = pg.indexrelname
    where i.tablename IN ('p4_lts_archive','p4_rep_ch_109','p4_rep_ch_109_rsp','p4_rep_ch_110_rsp','p4_rep_ch_121','p4_rep_ch_123','p4_rep_ch_123_rsp','p4_rep_ch_124','p4_rep_ch_125', 'p4_rep_ch_125_rsp','p4_rep_ch_126','p4_rep_ch_126_rsp','p4_rep_ch_127_rsp','p4_rep_ch_128','p4_rep_ch_128_rsp','p4_rep_ch_129','p4_rep_ch_129_rsp','p4_rep_ch_130','p4_rep_ch_131','p4_rep_ch_7' ,'p4_rep_ch_7_rsp' ,'p4_wd_check' ,'p4_wd_check_param' ) 
    order by i.tablename


La query SQL recupera le statistiche relative agli indici utilizzando la vista di sistema pg_stat_all_indexes. Analizziamo la query ed esploriamo ciascun componente:

* pg_relation_size(indexrelid): questa funzione restituisce lo spazio su disco utilizzato dall'indice specifico identificato da indexrelid.
* pg_total_relation_size(indexrelid): questa funzione restituisce lo spazio su disco totale utilizzato dall'indice specifico, incluse sia la dimensione dell'indice che la dimensione della tabella associata.
* sum(pg_relation_size(indexrelid)) Dimensione : la funzione somma calcola la dimensione totale di tutti gli indici sommando le dimensioni dei singoli indici.
* sum(pg_total_relation_size(indexrelid)) total_size: la funzione sum calcola la dimensione totale di tutti gli indici, comprese le dimensioni della tabella associata.
* sum(idx_scan) idx_scans: calcola il numero totale di scansioni dell'indice eseguite.
* sum(idx_tup_read) idx_reads: calcola il numero totale di tuple lette dagli indici.
* sum(idx_tup_fetch) idx_fetches: calcola il numero totale di tuple recuperate dagli indici.

bloat_percent: Questo calcola la percentuale di gonfiamento dell'indice sottraendo la dimensione dell'indice dalla dimensione totale, 
			dividendola per la dimensione totale (per ottenere la percentuale) e moltiplicandola per 100.

    SELECT pg_size_pretty(sum(pg_relation_size(indexrelid))) AS size,
    pg_size_pretty(sum(pg_total_relation_size(indexrelid))) AS total_size,
    sum(idx_scan) AS idx_scans,
    sum(idx_tup_read) AS idx_reads,
    sum(idx_tup_fetch) AS idx_fetches,
     (((sum(pg_total_relation_size(indexrelid)))-(sum(pg_relation_size(indexrelid)))) / ((nullif(sum(pg_total_relation_size(indexrelid)), 0)))) * 100 AS bloat_percent
    FROM pg_stat_all_indexes;


Nel momento in cui vogliamo calcolare quanto le dead tuple incidono sulle live tuple dobbiamo calcolare **incidenza percentuale** che consente di determinare in termini percentuale il rapporto tra un valore e l'altro. L'operazione rientra tra le cosidette 'formule percentuali inverse' in quanto la percentuale non è nota a priori ma in questo caso deve essere calcolata a partire dai valori

 		P=(V/N) * 100
  	il valore 10 rispetto a 200 è il 5% ad esempio.


![image](https://github.com/user-attachments/assets/9c07e212-6267-406a-b32f-625bbc3becf4)

Sottolineamo che nel momento in cui andiamo a confrontare le righe effettivamente presenti nella tabella e le live tuple, questi due valori potrebbero non combaciare per i seguenti motivi:
- MVCC(Multi-Version COncurrency Control): perchè fino a quando l'operazione di vacuum non è eseguita sia la nuova che vecchia versione esiste e questo può incidere un conteggio delle tuple attive maggiore rispetto alle righe effettivamente visibili;
- Autovacuum: se questo demone non è andato in esecuzione di recente il numero delle righe potrebbe non essere corretto;
- Statistics Collector Delay: contiene le statistiche delle live e dead tuple ma potrebbe avere informazioni non aggiornate. Lanciare un analyze potrebbe risolvere la problematica;
- Estimation: il numero riportato da pgstat_user_tables è una stima e non può sempre riflettere il numero estatto della tabella

## Soglie dead tuple

La soglia alla quale una percentuale elevata di dead tuples può causare problemi in PostgreSQL dipende da vari fattori, come la dimensione della tabella, la frequenza delle operazioni di modifica (inserimenti, aggiornamenti e cancellazioni) e le operazioni di VACUUM. 
Tuttavia, esistono alcune linee guida generali che possono aiutarti a capire quando la presenza di dead tuples diventa un problema significativo.

Indicazioni Generali sulla Percentuale di Dead Tuples
1. 0-5% di Dead Tuples:
	- In generale, una percentuale di dead tuples inferiore al 5% non dovrebbe causare gravi problemi alle prestazioni. PostgreSQL è in grado di gestire una quantità moderata di dead tuples senza impatti significativi.
	- In questo intervallo, il sistema di gestione dei vacuum regolare e le operazioni di VACUUM automatico di PostgreSQL possono mantenere le prestazioni relativamente stabili.
	
 2. 5-20% di Dead Tuples:
	- Quando la percentuale di dead tuples raggiunge una soglia tra il 5% e il 20%, è un buon indicatore che potrebbero esserci problemi di bloat (gonfiore) e che è il momento di eseguire un VACUUM più frequente o addirittura un REINDEX.
	- Le prestazioni potrebbero iniziare a risentirne, specialmente per le query che richiedono letture intensive delle tabelle con molte dead tuples.

3. Oltre il 20% di Dead Tuples:
	- Quando la percentuale di dead tuples supera il 20%, il bloat diventa un problema serio. A questo livello, PostgreSQL potrebbe avere difficoltà a mantenere prestazioni ottimali.
	- Le operazioni di lettura e scrittura possono diventare più lente a causa della necessità di più I/O per scavare attraverso le dead tuples.
	- È altamente consigliato eseguire un REINDEX, un VACUUM FULL o considerare il partizionamento della tabella se il bloat persiste.
	
 4. Soglie superiori al 50%:
	- Se la percentuale di dead tuples supera il 50%, potresti riscontrare un serio degrado delle prestazioni, specialmente nelle tabelle con un grande numero di righe. Questo è un segnale di un accumulo di dead tuples che non è stato gestito correttamente, anche con il VACUUM.
	- In questi casi, potrebbe essere necessario eseguire una manutenzione più aggressiva, come VACUUM FULL o una ristrutturazione della tabella.

Altri Fattori da Considerare:
- Frequenza di VACUUM: PostgreSQL ha un processo di autovacuum che gestisce automaticamente la rimozione delle dead tuples. Tuttavia, se il carico del database è molto elevato o se le operazioni di aggiornamento e cancellazione sono frequenti, potresti dover eseguire manualmente dei VACUUM più frequenti.
- Dimensione della Tabella: Le tabelle più grandi sono più suscettibili a problemi legati alle dead tuples, e potrebbero richiedere un monitoraggio regolare.
- Risoluzione del problema: Se il bloat è un problema, potresti considerare altre tecniche di manutenzione, come il partizionamento delle tabelle, l'uso di indici efficienti o la gestione del carico di lavoro per ridurre il numero di operazioni di aggiornamento e cancellazione.




