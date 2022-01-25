# Progetto Sogni d'oro
## Background
Il problema richiede di simulare l'associazione di un insieme di clienti con un insieme di hotel variando le politiche di associazione e vedere l'impatto su alcuni KPI fondamentali come il volume di business generato e il livello di soddisfazione generato.

Ogni cliente ha la possibilità di inserire un numero arbitrario di preferenze di hotel, il primo hotel che inserisce avrà un livello maggiore di soddisfazione e così via fino all'ultimo. 

I clienti sono ordinati sulla base dell'ordine di arrivo.

Venogno poi date altre informazioni: 
- lo sconto da applicare al singolo cliente al prezzo della camera dell'hotel a cui viene associato
- il prezzo di una camera di hotel
- la disponibilità di camere

Vengono poi definite una serie di possibili strategie / scenari che possono essere testati:

1. assegnazione casuale:
    - ogni cliente viene assegnato casualmente ad un hotel
    - ogni cliente viene assegnato casualmente ad un hotel rispettando le sue preferenze (ma senza seguire un ordine)
    - ogni cliente viene assegnato casualmente ad un hotel rispettando l'ordine delle sue preferenze
2. assegnazione rispetto l'ordine di arrivo del cliente: ogni cliente viene associato ad un hotel sulla base dell'ordine di arrivo e l'ordine delle sue preferenze
3. assegnazione rispetto al prezzo degli hotel: ogni cliente viene associato all'hotel nelle preferenze con prezzo più basso
3. assegnazione rispetto al capienza degli hotel: ogni cliente viene associato all'hotel nelle preferenze con maggior capienza

Per ogni strategia è possibile dire se si vuole assegnare casualmente i clienti a cui non è stato possibile assegnare un hotel.

## Struttura del progetto
Il progetto si compone di 3 parti fondamentali:
1. Esplorazione dei dati sorgenti
2. Implementazione delle classi
3. Simulazione e analisi dei risultati

## Esplorazione dei dati sorgenti
L'esplorazione dei dati viene fatta in due notebook distinti:
1. [explore_data.ipynb](./explore_data.ipynb): viene fatta un'analisi prettamente qualitativa dei dati (presenza di valori NaN, duplicazione di chiavi, range di variazione dei dati,...)
2. [EDA.ipynb](./EDA.ipynb): viene fatta un'analisi più quantitativa andando ad analizzare in dettaglio le distribuzioni di alcune dimensioni di interesse e capire se è possibile stabilire dei pattern di scelta tra i clienti

### Analisi qualitativa

Le analisi hanno portato alla identificazione di un errore nel file di preferences in cui era possibile che uno stesso cliente potesse dichiarare nelle preferenze lo stesso hotel.

Lo script va a rigenare la giusta associazione tra guest-hotel.

Non sono state rilavati altri problemi nei dati.

### Analisi quantitativa

L'analisi quantitativa invece mira appunto a dare un primo senso ai dati, capire le varie relazioni tra i dati in particolare se è possibile identificare dei pattern di scelta da parte dei singoli (ad esempio la propensione di alcuni guest a indicare hotel con fasce di prezzo alte o basse,...).

Quello che possiamo dire dopo un'attenta analisi è che non c'è un comportamento di scelta sistematico da parte dei clienti, si osserva una sorta di indifferenza del cliente al prezzo dell'hotel.

Il dettaglio delle analisi viene lasciato alla presentazione.