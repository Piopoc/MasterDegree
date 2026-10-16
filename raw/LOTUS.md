Il paper **"Semantic Operators and Their Optimization: Enabling LLM-Based Data Processing with Accuracy Guarantees in LOTUS"** (Patel et al., PVLDB 2025) introduce un nuovo paradigma per l'elaborazione semantica di grandi moli di dati (bulk semantic processing) integrando i modelli di linguaggio (LLM) con le trasformazioni relazionali.

Di seguito sono delineati i **concetti fondamentali** descritti nella fonte:

---

### 1. Motivazione e Limiti degli Approcci Esistenti

- **Fallimento dei metodi tradizionali**: La ricerca SQL con tabelle esplicite richiede un pre-processing esaustivo di tutte le categorie possibili. Il keyword matching fallisce di fronte ai sinonimi e non garantisce la rilevanza semantica del contesto. L'Information Retrieval (IR) e gli embedding producono un ranking continuo, ma non forniscono una separazione netta per decidere cosa selezionare o scartare.
- **Limiti delle pipeline procedurali**: Scrivere codice procedurale con LLM richiede di gestire manualmente l'iterazione sulle righe, la costruzione dei prompt, la gestione degli errori e l'ottimizzazione di costi e latenze.

---

### 2. Il Modello degli Operatori Semantici

- **Definizione**: Un operatore semantico è una trasformazione dichiarativa su uno o più dataset parametrizzata da un'espressione in linguaggio naturale (_langex_).
- **Separazione delle responsabilità**: Disaccoppia la logica applicativa (_cosa si vuole ottenere_) dalla logica di esecuzione AI (_come i modelli vengono invocati_).
- **Espressione in linguaggio naturale (_langex_)**: L'operatore viene parametrizzato inserendo i nomi degli attributi tra parentesi graffe (ad esempio `{plot}`) all'interno del predicato.
- **Algoritmo di riferimento (_Reference Algorithm_)**: Poiché esistono diverse strategie di esecuzione, il comportamento corretto dell'operatore è formalmente definito rispetto a un algoritmo di riferimento ad alta qualità.

---

### 3. La Suite degli Operatori Semantici (Toolbox)

Il lavoro estende le classiche trasformazioni relazionali con capacità di ragionamento semantico:

- **`sem_filter`**: Mantiene solo le righe che soddisfano il predicato espresso nella _langex_.
- **`sem_join`**: Unisce due tabelle valutando un predicato espresso in _langex_ su coppie di righe appartenenti alle due relazioni (con complessità di riferimento \(O(|T_1| \cdot |T_2|)\)).
- **`sem_topk`**: Restituisce le prime \(k\) righe che soddisfano al meglio un criterio di ordinamento semantico.
- **`sem_agg`**: Aggrega e combina le informazioni di più righe in un unico risultato sintagmatico.
- **`sem_group_by`**: Raggruppa le righe in categorie semantiche descritte dalla _langex_.
- **`sem_map`**: Trasforma o proietta ciascuna riga in modo indipendente secondo le istruzioni specificate.

---

### 4. Ottimizzazione con Garanzie Statistiche di Accuratezza

Invocare un LLM di grandi dimensioni su ogni riga (\(O(N)\) per i filtri) o su ogni coppia (\(O(N \cdot M)\) per i join) è estremamente costoso. L'ottimizzazione riduce i costi computazionali fornendo **garanzie statistiche di accuratezza** (target di precisione e recall \(\gamma\) e livello di confidenza \(1 - \delta\)) rispetto all'algoritmo di riferimento:

- **Cascata per `sem_filter`**:
    1. Utilizza un modello _helper_ più piccolo ed economico per calcolare un _proxy score_ basato sulle log-probabilità dei token.
    2. Ricalibra i punteggi in quantili nell'intervallo \(\).
    3. Applica un campionamento (di importanza e uniforme) per selezionare un sottoinsieme da valutare con il modello principale (_oracolo_).
    4. Stima e adegua statisticamente una soglia di rigetto \(\tau^-\) e una soglia di accettazione \(\tau^+\), tenendo conto dell'incertezza statistica.
    5. Scarta o accetta automaticamente le tuple al di fuori dell'intervallo ed invoca l'oracolo principale solo per le tuple incerte comprese tra \(\tau^-\) e \(\tau^+\).
- **Approssimazione per `sem_join`**: Evita il costo quadratico stimando i proxy score tramite similarità tra embedding con due strategie: _Sim-Filter_ (similarità diretta) o _Project-Sim-Filter_ (proiezione intermedia tramite LLM seguita da similarità di embedding).

---

### 5. Il Framework LOTUS

- **LOTUS** (**LLMs Over Tables of Unstructured and Structured Data**) è il sistema che implementa gli operatori semantici fornendo un'API stile **Pandas DataFrame** in Python.
- Consente di concatenare gli operatori semantici in pipeline dichiarative (ad esempio `movies.sem_filter(...).sem_topk(...).sem_agg(...)`), lasciando che l'infrastruttura gestisca l'esecuzione e l'ottimizzazione dei modelli sottostanti.