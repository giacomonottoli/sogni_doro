# Progetto Sogni d'oro
## Background
Il problema richiede di simulare l'associazione di un insieme di clienti con un insieme di hotel variando le politiche di associazione e vedere l'impatto su alcune statistiche fondamentali come il volume di business generato e il livello di soddisfazione generato.

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
2. Implementazione delle strategie
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

## Implementazione delle classi
### Obiettivi

Il progetto richiede di implementare le seguenti strategie di allocazione:
- Casuale: i clienti sono distribuiti casualmente nelle stanze fino a esaurimento dei posti o dei clienti
- Preferenza cliente: i clienti sono distribuiti nelle stanze in ordine di prenotazione (il numero del cleinte ne indica l'ordine) e attribuendogli l'hotel in ordine di preferenza, fino a esaurimento dei posti o dei clienti
- Prezzo: i posti in hotel sono distribuiti in ordine di prezzo, partendo dall'hotel più economico e in subordine in ordine di prenotazione e preferenza fino a esautimento posti o clienti
- Disponibilità: i posti in hotel sono distribuiti in ordine di disponibilità di stanze, partendo dall'hotel più capiente e in subordine in ordine di prenotazione e preferenza fino a esautimento posti o clienti

Viene poi richiesto il calcolo delle seguenti statistiche:
- il numero di clienti sistemati
- il numero di stanze occupate
- il numero di hotel diversi occupati 
- il volume complessivo di affari (guadagno complessivo di ogni hotel)
- il grado di soddisfazione dei clienti


### Modellizzazione del problema

L'attribuzione di un guest ad un hotel può essere modellata come:

\begin{align*}
[a_{ij}] &\text{ con } i = 1, \dots, n \text{ (numero di guest) e } j = 1, \dots, m \text{ (numero di hotel)}\\
&\text{ e } a_{ij} \in \{ 0, 1 \}
\end{align*}

La matrice risultante avrà 1 laddove il guest i viene associato al hotel j altrimenti avrà 0


Vengono dati anche i seguenti vincoli:

\begin{equation}
    \begin{cases}
\sum_{j=1}^{m} a_{ij} &\leq 1\quad \forall \,j = 1, \dots, m\\\\
\sum_{i=1}^{n} a_{ij} &\leq r_j\quad \forall \,i = 1, \dots, n \text{ dove } r_j \text{ è la disponibilità di camere dell'hotel j}\\\\
a_{ij} &\in \{ \pmb{y_i}\}\, \text{ dove } \pmb{y_i} \text{ è il vettore contenente le preferenze del i-esimo cliente}
\end{cases}
\end{equation}


### Schema Generale

Da un punto di vista concettuale i due blocchi principali da realizzare sono:
- definizione delle strategie di allocazione dei guest
- definizione delle metriche risultanti

Si è scelto di atrarre il problema con una serie di classi in cui venga allocato:
1. nella classe base tutti i metodi in comune a tutte le strategie di allocazione come ad esempio l'inizializzazione delle variabili, il metodo di assegnazione dei guest agli hotel e la produzione / visualizzazione delle metriche
2. una serie di classi derivate, in cui vengono implementate le singole strategie di allocazione (una classe per ognuna di esse)

Il principio è quello di definire una volta sola il meccanismo di assegnazione guest-hotel nella classe base e lasciare alle classi derivate la definizione delle priorità legate all'assegnazione dei guest o degli hotel.

Questo permette da un punto di vista pratico di avviare l'allocazione con un unico metodo per tutte le classi (metodo .assign()) e al contempo avendo una classe per ogni strategia permette di mantenere massima flessibilità nel definire funzioni e parametri aggiuntivi di valutazione nelle singole classi derivate.

Il codice è stato implementato all'interno del file [allocation_strategy.ipynb](./allocation_strategy.ipynb)

La classi definite sono:
- classe base: BaseAllocation()
- classe derivate:
    - RandomGuestAllocation()
    - OrderGustAllocation()
    - PriceHotelAllocation()
    - AvailabilityHotelAllocation()
    
### BaseAllocation

La classe permette:
1. inizializzazione delle variabili
2. meccanismo di allocazione del guest al hotel
3. elaborazione dei risultati ottenuti
4. calcolo delle statistiche richieste

Argomenti in input:
- tabella iniziale dei guest (<span style="color:orange">*df_guests*</span>)
- tabella iniziale degli hotel (<span style="color:orange">*df_hotels*</span>)
- tabella iniziale delle preferenze (<span style="color:orange">*df_preferences*</span>)
- rilassa vincolo delle allocazioni per preferenza (<span style="color:orange">*assign_all*</span>)


Si è scelto di settare la classe come abstract class in quanto l'obiettivo è di raccogliere al suo interno le operazioni delle altre sottoclassi. Tale classe non potrà essere quindi inizializzata in quanto non presenta tutti i metodi necessari per poter eseguire l'allocazione 

#### 1. Inizializzazione delle variabili

Metodi:
- *\__init__()*

In questa parte, oltre alla lettura delle tabelle in input (tabella delle prefenrenze, tabella dei guest e tabella degli hotel) e alla inizializzazione delle statistiche vengono create le due matrici:

- una matriche delle preferenze guest-hotel
- una matrice degli hotel per mantenere aggiornato il numero di camere occupate

La matrice delle preferenaze (contenente nella fase di inizializzazione tutti 0) è una matrice \[*n* x *m*\] dove n è appunto il numero di guest e m è il numero di hotel. 

L'indice di riga rappresenta la posizione del guest letta nella tabella dei guest. Ad esempio il primo guest avrà indice 0, il secondo guest posizione 1, ..., l'ultimo guest avrà posizione n-1.

Analogalmente l'indice di colonna rappresenta la posizione del hotel nella tabella dei guest. Ad esempio il primo hotel avrà indice 0, il secondo hotel posizione 1, ..., l'ultimo hotel avrà posizione m-1.

La matrice degli hotel ha come prima colonna il numero di camere disponibili, nella seconda colonna il prezzo delle camere e nella terza colonna il numero di camere occupate.

Va ricordato che in base ai vincoli la somma per riga della matrice delle preferenze deve essere minore o uguale a 1 e la somma per colonna deve essere minore o uguale della disponibilità di camere del dato hotel

#### 2. Meccanismo di allocazione del guest al hotel

Metodi:
- *assign()*
- *_access_pref_matrix()*
- *_define_guest_order()* \[abstractmethod\]

In questa parte i metodi elencati implementano l'allocazione dei guest sulla base della strategia di allocazione. 

La strategia viene appunto definita nelle classi derivati andando a definire in modo combinato un ordine di estrazione dei guest e un ordine di estrazione degl hotel. La definizione avviene andando a definire nelle classi derivate il metodo \_define_guest_order(). Per questo motivo abbiamo ritenuto opportuno decorare questa funzione con abstractmethod

Il metodo *assign()* prende da un dizionario (variabile <span style="color:lightblue"> *self.pref_by_guest*</span>) secondo un dato ordine di estrazione dei guest (variabile <span style="color:lightblue"> *self.guest_order*</span>) le potenziali coppie di allocazione (guest, hotel) secondo un dato criterio di allocazione.

Per ogni combinazione (guest, hotel) si valuterà se l'hotel ha camere disponibili e se: 
- sì mettere 1 nella matrice delle preferenze e aggiorna il numero di camere assegnate per quel dato hotel
- altrimenti passare ad un altra combinazione (guest, hotel) da testare.

L'allocazione finisce quando:
- tutti i guest sono stati allocati
- tutte le camere sono state occupate
- non è più possibile allocare guest nelle camere rimaste

Sì è data la possibilità con l'argomento <span style="color:orange"> *assign_all*</span> di rilassare il vincolo relativo alle preferenze e di poter assegnare quei guest, che non hanno avuto un hotel assegnato, ad hotel simili per livello di prezzo a quelli dichiarati nelle preferenze.

Ad esempio se un guest non allocato con preferenze in \[hotel_1, hotel_2, hotel_3\] si va ad assegnare il guest a quell'hotel disponibile che ha il prezzo più vicino a uno qualsiasi di quelli dichiarati. A parità di delta si sceglie quello con preferenza maggiore.

#### 3. Elaborazione dei risultati ottenuti
Metodi:
- *_preprocessing_results()*

La procedura ha lo scopo di trasformare la matrice delle preferenze \[*n* x *m*\] in una tabella flat in cui ogni riga viene riportato l'assegnazione (guest, hotel).

Per fare questa operazione si è utilizzato la funzione numpy *argmax()* il cui scopo è quello di trovare l'indice della colonna con valore 1 (hotel scelto). Nel caso in cui il guest non abbia un hotel assegnato la funzione restice il valore 0 (prima colonna che incontra con il valore massimo). Per evitare questo problema si è aggiunto una colonna di 1 alla fine della matrice, in questo modo la funzione restituisce un valore di indice superiore al numero degli hotel realmente disponiboli.

#### 4. Calcolo delle statistiche richieste
Metodi:
- *_compute_stats()*
- *\__str__()*
- *_partial_hotel_booking_ratio()*

Tali metodi concorrono in vario titolo alla produzione e visualizzazione dei risultati prodotti dalle singole startegie di allocazione.

In particolare *_compute_stats()* può essere chiamata solo dopo aver chiamato il me